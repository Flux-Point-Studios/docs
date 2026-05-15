---
description: M-of-N committee-attested settlement of Cardano-anchored DeFi intents on Materios
---

# Intent Settlement

Materios runs a permissionless M-of-N committee that **autonomously settles Cardano-anchored DeFi intents** without a central keeper, batcher, or trusted middleman.

A user posts a Cardano transaction containing the settlement evidence. The committee — running open-source cert-daemons on independent nodes — observes the Cardano L1 tx, cross-checks 8 falsifiable facts, signs the canonical settlement digest, peer-shares signatures, and lands one M-sig `attest_settle` extrinsic on Materios. Sub-10s from request to settled, end-to-end cryptographically verifiable, zero manual intervention.

It is the first production Cardano L2 path where the settlement decision is made by independent committee observation of L1, not by a single batcher or a single archive node.

***

## Why This Exists

DeFi settlement on Cardano today depends on one of:

1. **A single batcher** (Minswap, SundaeSwap pre-Gummiworm) — the operator decides which orders settle, in what order, at what price. Reorgs, BEV, and frontrun risk all concentrate on one party.
2. **A Hydra head** — multiparty channel that requires all participants to be online for every state transition. Strong cryptography, weak liveness when one party drops.
3. **A bridged sidechain** — Materios's category. Trust depends on who watches L1, who signs the settlement, and how falsifiable the chain of custody is.

Materios's intent-settlement pallet inverts the failure modes of category (3):

| Centralized batcher | Hydra head | **Materios intent-settlement** |
|---|---|---|
| One operator decides settlement | All-or-nothing liveness | M-of-N committee, any N-M can be offline |
| No cryptographic provenance | Cryptographic, but private | Public on-chain, **every sig verifies sr25519 against canonical bytes the pallet rebuilds from chain state** |
| Operator can frontrun | No BEV (only participants) | Committee cannot frontrun — they sign chain-state-derived digests, not orderings |
| No skin in the game | No external slashing | Slashable MATRA bond per attestor (#286 follow-on shipping) |

The committee's job isn't to **decide** — it's to **observe and confirm**. The pallet defines exactly which Cardano L1 fact must be true for the settlement to be valid; the committee's signature is just a multi-party witness that the fact is true. Anyone running a follower node can refute a forged settlement.

***

## How It Works

The flow has two on-chain phases plus a permissionless intermediate step:

```
   Phase 1: PIN THE CLAIM
   ─────────────────────
1. User (or anyone) submits a Cardano L1 tx that pays the beneficiary
   the voucher amount.
2. Anyone (typically the beneficiary or a keeper) calls
       IntentSettlement::request_settle(claim_id, cardano_tx_hash,
                                         observed_at_depth, observed_slot)
   which PINS the Cardano evidence into ClaimSettlementRequests storage.
   The pallet validates observed_at_depth >= MinFinalityDepth and
   observed_at_depth fits in u32.

   Phase 2: COMMITTEE ATTESTATION (autonomous)
   ────────────────────────────────────────────
3. Each cert-daemon polls ClaimSettlementRequests every 12s.
4. For each pending request, the daemon INDEPENDENTLY:
       a. queries Kupo for the Cardano tx → outputs + slot + block hash
       b. resolves block height via Ogmios chain-sync mini-protocol
          (header_hash → block height) — see PR #31
       c. cross-checks 8 facts (see "The Eight Facts" below)
       d. computes the canonical STCA preimage from chain state
          (213 bytes, byte-exact match against the pallet)
       e. signs blake2_256(preimage) with its sr25519 committee key

5. The daemon POSTs its (pubkey, sig, digest) to the gateway's
       /v2/multisig_sigs/{kind}/{key}
   bulletin board. Gateway verifies sr25519 before storing.

6. The daemon GETs peer sigs filtered to its locally-computed digest
   (defense-in-depth — peer sigs over a different digest are dropped
   client-side even if the gateway misbehaves).

7. When >= MinSignerThreshold sigs are assembled, the daemon submits
   one M-sig envelope:
       IntentSettlement::attest_settle(claim_id, [(pubkey, sig), ...])

8. The pallet's `ensure_threshold_signatures` re-runs sr25519 verify on
   every sig against the digest it rebuilds from chain state. First
   successful submission emits:
       IntentSettlement::ClaimSettled {
         claim_id, cardano_tx_hash, settled_direct
       }
   Concurrent submissions from other daemons bounce with
   `SettlementRequestMissing` (already-consumed → idempotent).
```

***

## The Eight Facts

Every cert-daemon must INDEPENDENTLY confirm all eight before signing. A refusal on any fact is logged with the specific reason and is the central safety property — auditors grep `settle_attestor: REFUSE` to find any committee member who disagreed with a request.

| # | Fact | Sourced from |
|---|---|---|
| 1 | The Cardano tx exists | Kupo `/matches?transaction_id=` |
| 2 | observed_at_depth ≥ MinFinalityDepth blocks | Ogmios chain-sync resolution |
| 3 | observed_slot matches Kupo's slot for the tx | Kupo |
| 4 | amount_lovelace to beneficiary matches voucher | Kupo + chain state |
| 5 | mainchain_genesis_hash pins preprod vs mainnet | Ogmios networkMagic table |
| 6a | voucher.amount_lovelace matches request | Vouchers[claim_id] |
| 6b | voucher.beneficiary_addr_hash matches request | Vouchers[claim_id] |
| 7 | live chain_id (Materios genesis) matches request | Materios RPC |
| 8 | voucher_digest derived from chain state matches | Vouchers[claim_id] |

The pallet does NOT trust the daemon's word on any of these — it independently rebuilds the canonical STCA preimage from `ClaimSettlementRequests[claim_id]` + `Vouchers[claim_id]` and verifies every committee sig against the resulting digest. Daemons signing over the wrong digest get `InvalidSignature` and the settlement does not land. This is the "two-phase B+D" architecture from audit-P0 task #266.

***

## Live Status

Production on Materios preprod as of **2026-05-15**.

| Component | Status |
|---|---|
| spec-220 (settle_claim L1 verification) | LIVE since 2026-05-15 |
| spec-221 (expire_policy_mirror B+D) | LIVE since 2026-05-15 |
| Cert-daemon M-of-N envelope coordination | LIVE since 2026-05-15 |
| First fully-autonomous ClaimSettled | Block **#200660**, claim `0xe08c94624cdc3991…`, Cardano tx `bb59881af429ad09…` |
| Active committee members (preprod) | 2 (Gemtek + Node-3); MinSignerThreshold = 2 |
| Committee MaxSize | 64 |

### What "autonomous" actually means

In one Materios block before tip moved, this happened with no human or script in the loop after `request_settle`:

```
16:40:00.472  Node-3 daemon : assembled 2/2 sigs for e08c94624cdc3991... — submitting envelope
16:40:00.???  Materios block #200660 produced with the envelope inside
              IntentSettlement::ClaimSettled emitted
16:40:04.703  Gemtek daemon : assembled 2/2 sigs for same claim — submitting
16:40:06.017  Gemtek daemon : SettlementRequestMissing (already consumed — correct)
```

Two committee members independently observed the same Cardano L1 facts, computed byte-identical STCA digests, peer-shared sigs through the gateway bulletin board, and both submitted M-sig envelopes within 4 seconds of each other. The first to land won, the second got the expected idempotent refusal.

***

## Cryptographic Chain of Custody

The thing that makes a Materios settlement un-forgeable is the chain that ties the signed claim back to four independent sources of truth.

```
1. Cardano L1 transaction          ─── observed via Kupo+Ogmios
        │                              (independent followers per attestor)
        ▼
2. Materios on-chain Voucher       ─── stored in Vouchers[claim_id]
        │                              by the M-of-N issuance pallet
        ▼
3. Pinned settlement evidence       ─── ClaimSettlementRequests[claim_id]
        │                              with (cardano_tx_hash, observed_at_depth,
        │                              observed_slot, mainchain_genesis_hash)
        ▼
4. Canonical STCA digest             ─── blake2_256(213-byte preimage)
        │                              rebuilt by the pallet from (2)+(3)+chain_id
        ▼
5. M-of-N sr25519 envelope            ─── each sig must verify against (4)
                                         OR the pallet rejects entire submit
```

Every level is verifiable on chain. Forge any one and the verification chain breaks. The committee's sigs **commit to a 213-byte canonical structure containing every chain-state-derived field** — they cannot be replayed onto a different claim, a different Cardano tx, or a different network.

***

## What This Unlocks

Spec-220's settle path is the bottom-of-stack primitive that enables everything Materios will ship in 2026:

| Layer | Status | Depends on |
|---|---|---|
| **Settle claim** (spec-220) | LIVE 2026-05-15 | (this page) |
| **Expire policy** (spec-221) | LIVE 2026-05-15 | spec-220 committee path |
| **MM rebate program v0** | Design locked, impl queued | spec-220 attestation |
| **Perp engine v0** | Design locked, impl queued | spec-220 attestation + Aegis price oracle |
| **Materios Oracle Network Phase 1** | Design locked, impl queued | spec-220 committee + Aegis publishers |
| **Compute portal Wave 3 (heterogeneous TEE)** | LIVE on attestation pallet | spec-220 committee path |

Every product line above re-uses the same committee, the same canonical-digest pattern, the same gateway aggregator, the same on-chain verification primitive. **One M-of-N path; many DeFi flows.**

***

## Comparison Frame

| | Materios intent-settlement | Hydra (Cardano L2) | Gummiworm (Sundae L2) | Generic bridge |
|---|---|---|---|---|
| Settlement decision | M-of-N committee observation | All participants online | Off-chain archive | Custodian / federation |
| Plutus parity | No (Substrate runtime) | Yes | Yes (planned) | N/A |
| Cardano L1 anchor | Periodic checkpoint (CIP-31 metadata) | Every state update | Periodic archive | Variable |
| TEE attestation | Optional (Wave 3 pallet, LIVE) | None | None | None |
| MEV / BEV exposure | Committee cannot front-run | None | TBD | Custodian-dependent |
| Throughput baseline (live) | ~250 TPS measured, 10k target | <100 TPS | TBD | Variable |
| Autonomous settle witness | YES (this page) | N/A (participants ARE the witnesses) | TBD | NO |

Materios is **not** a Hydra replacement — Hydra wins the pure-Plutus story and the low-latency story for whitelisted participant sets. Materios wins the **production-grade, attested, permissionless-throughput-without-Plutus-parity** story for intent flows where a Substrate-based runtime is acceptable.

***

## Architecture

### Pallets

- **`pallet-intent-settlement`** — owns `Vouchers`, `ClaimSettlementRequests`, `Intents`, `PolicyExpireRequests`. The `attest_settle` and `attest_expire_policy` extrinsics each require M-of-N committee sigs in a single envelope. No cross-call accumulation; threshold checked in one shot. (spec-220, spec-221)
- **`pallet-orinq-receipts`** — provides the M-of-N committee membership + sr25519 sig verifier used across all settlement paths. (Live since spec-219.)
- **`pallet-tee-attestation`** — vendored Acurast verifier for the Witness Network. Re-uses the same committee. Genesis kill-switch off; ARM TrustZone path live since 2026-05-13.

### Off-chain services

- **`services/blob-gateway`** — TypeScript Express server. New `/v2/multisig_sigs/{kind}/{key}` bulletin board (`kind` ∈ `settle` | `expire`) brokers sig sharing between cert-daemons. sr25519 verify on every POST, 24-hour TTL, defense-in-depth digest filter on GET.
- **`daemon/` (operator-kit)** — Python cert-daemon. Two new modules: `settle_claim_attestor.py` (spec-220 path) and `expire_policy_attestor.py` (spec-221 path). Both share the same multisig_aggregator client and submit_*_envelope methods.

### Trust assumptions

The committee threshold is governance-set. With M=2 today on a 2-active committee (Gemtek + Node-3), both must agree for any settlement to land. As the committee grows to N=4 (MacBook re-registration + Hetzner SPO pending), M=2 becomes 2-of-4 and gains 2 fault-tolerance. The cryptoeconomic gate per attestor (slashable bond) lands in the v2 rewards/slashing wave (#286 follow-on).

***

## Running a Committee Daemon

The cert-daemon is open source. Anyone can run it; the on-chain pallet is the gate:

1. Operator deploys `materios-operator-kit` cert-daemon container (image at `ghcr.io/flux-point-studios/materios-operator-kit:7c88e0456b6941030af4619b578b0a41ec3d1ed0` or later). Sets `BLOB_GATEWAY_URL` to point at the public Materios blob gateway.
2. Operator registers as a partner-chain candidate on Cardano L1 (IOG Partner Chains pallet flow). On Ariadne selection, their pubkey joins `CommitteeMembers`.
3. Operator funds a Kupo + Ogmios pair pointing at their own Cardano node. (Public Materios infrastructure includes a fallback, but independent observation is the point.)
4. Daemon starts polling, signing, peer-sharing. Refusals are logged at WARNING with the specific fact mismatch — operators can grep `settle_attestor: REFUSE` to audit any disagreement.

No bond, no whitelist, no committee admission tx today. The single gate is the partner-chain candidate selection on Cardano L1 — the same mechanism that gates Materios block production. As mainnet approaches, a slashable MATRA bond per attestor will be the second gate.

***

## Roadmap

| Phase | Quarter | Deliverable |
|---|---|---|
| **Spec-220** (settle path) | Live | Committee-attested settle, 8-fact verification, M-of-N envelope, Cardano L1 anchor |
| **Spec-221** (expire path) | Live | Symmetric attested-expire for stale intents |
| **MON Phase 1** (oracle rail) | Q2 2026 | Aegis publishers post prices to Materios pallet-oracle as a second output rail |
| **Perp engine v0** | Q3 2026 | i128 positions, premium-index funding, pull-based oracle |
| **MM rebate program v0** | Q3 2026 | 5M MATRA over 24 months, bonded-permissionless |
| **Slashable attestor bond** | Q3 2026 | Cryptoeconomic gate per committee member |
| **Cross-chain intent dApp** | Q4 2026 | First-party SDK for posting intents + watching settlements |

***

## On-chain Receipts

Verifiable today on preprod:

- **First autonomous ClaimSettled**: claim `0xe08c94624cdc3991b7715a7f7f5030e1946afb5187c8f67f29ef6898f00c2992` at Materios block **#200660**, Cardano tx `0xbb59881af429ad09ca0edead5a8d93acb37decabf83d6008f600fefbea2a8664`
- **Earlier same-day demos** (manual aggregator): claims `0xf9ef6b1f…`, `0xbab7a053…`, `0xfeb61d09…`, `0x2a675332…`
- **Spec-220 + spec-221 ceremonies** executed via multisig sudo 2-of-3 the same day

Anyone with a Materios RPC connection can independently verify every settlement: the pallet's rebuilt STCA digest, the M-of-N sigs, the Cardano tx existence at the pinned depth + slot, and the byte-exact `cardano_tx_hash` match. Nothing is taken on trust.

***

## Getting Involved

| You are a… | Next step |
|---|---|
| Cardano DeFi project | Open an issue at https://github.com/Flux-Point-Studios/materios with your intent shape — we'll work with you to map it onto the pallet |
| SPO / validator | Run a cert-daemon. See [SPO Onboarding](spo-onboarding.md) |
| Researcher / auditor | Read the spec-220 design memo + the on-chain pallet code at `pallets/intent-settlement` — the canonical STCA preimage is byte-pinned, the verification flow is one function |
| Integrator | The off-chain SDK is `@fluxpointstudios/orynq-sdk-payer-materios-x402` — pay-per-use on the existing 402 middleware |
| Investor / partner | hello@fluxpointstudios.com — the M-of-N committee is the substrate every Materios product compounds on |

The intent-settlement pallet is the keel. Everything Materios ships in 2026 is a sail on it.
