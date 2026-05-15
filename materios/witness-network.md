---
description: TEE-attested distributed uptime + probe network on consumer Android phones
---

# Materios Witness Network

The **Materios Witness Network** is a decentralized network of consumer Android phones that produce TEE-rooted attestations of off-chain facts — uptime probes, URL availability, certificate validity, version drift — and anchor every observation to the Materios chain.

Every observation is signed inside the phone's hardware-isolated secure element (Android KeyMint / StrongBox). The signing key's attestation challenge cryptographically commits to the observation payload, so the on-chain signature is **un-forgeable proof that this specific result came out of this specific device at this specific moment**.

It is the first public network where a $200 phone is a load-bearing piece of cryptographic infrastructure.

***

## Why This Exists

Centralized uptime monitoring (Pingdom, UptimeRobot, StatusCake) has three structural problems:

1. **A single operator decides what "up" means.** If their probes see a 200 and yours don't, the operator wins.
2. **No cryptographic provenance.** A status page screenshot is unfalsifiable in either direction.
3. **No skin in the game.** A monitor that misreports has no economic consequence.

The Witness Network inverts all three:

| Problem | Centralized monitor | Materios Witness Network |
|---|---|---|
| Who decides "up"? | The operator | M-of-N independent phones |
| Cryptographic provenance | None | Android Key Attestation chain rooted at Google's hardware CA |
| Skin in the game | None | Slashable MATRA bond per attestor (v2) |

The same Android Key Attestation chain that banks use to gate sensitive operations is what gates a Witness Network attestor. A divergent attestation is provable on chain, in real time, by anyone.

***

## How It Works

```
Real-world event (URL probe, cert check, version fetch)
    │
    ▼
Phone's WitnessWorker executes the probe
    │ (HTTPS GET / DNS lookup / TLS handshake)
    ▼
Result + timestamp committed as Android KeyMint attestation challenge
    │ (KeyMint generates a fresh key whose attestation cert chain
    │  includes the challenge bytes — proof the result is bound to
    │  this exact key, which is bound to this exact device)
    ▼
Result + cert chain POSTed to Materios blob gateway
    │
    ▼
Gateway verifies the Android Key Attestation chain → Materios
    │ (root CA: Google; leaf: this device's KeyMint key)
    ▼
On-chain event: TeeAttestation::EvidenceVerified
    │ (binds the probe result to the device's attest_key_hash)
    ▼
Periodically anchored to Cardano L1 via the Materios checkpoint cadence
    │
    ▼
Anyone can verify, no trusted intermediary
```

***

## Live Status

The Witness Network is live on Materios preprod as of **2026-05-13**. Current state:

| Component | Status |
|---|---|
| First TEE-attested probe on chain | Block **#179467** (Pixel StrongBox), block **#180364** + #180370 + #180371 (Moto G TEE) |
| KeyMint signing algorithms supported | sr25519 (envelope), ed25519, **secp256r1 (KeyMint native, P-256, live 2026-05-14)** |
| APK pairing | Self-registration via Android attestation cert chain; zero-touch onboarding |
| Probe targets (today) | `materios.fluxpointstudios.com`, `github.com`, `cardano.org` — configurable via gateway `/witness/targets` endpoint |
| Witness landing + dashboard | https://fluxpointstudios.com/materios/witness |
| Pallet on chain | `pallet-tee-attestation`, kill-switch OFF, multi-arch verifier (ARM TrustZone + AMD/Intel SGX stubs) |

### What "live on chain" actually means

For the Moto G witness (`attest_key_hash = 0x0c4c6f81…`) the event chain in `IntentSettlement` storage looks like this:

```
TeeAttestation::EvidenceVerified {
  attestor: <Moto G's sr25519 envelope key>,
  attest_key_hash: 0x0c4c6f81…,
  evidence_type: arm_trustzone,
  composite_trust_score: 1,
}
```

And in `pallet-tee-attestation::CompositeTrustScores`:

```
CompositeTrustScores[0x0c4c6f81…] = 1
```

That's the on-chain proof that the device is a real ARM TrustZone phone, anchored to Google's root attestation CA, with no human-introduced trust. Any wallet, any explorer, any reader can verify.

***

## Cryptographic Chain of Custody

The thing that makes a Witness attestation un-spoofable is the chain that ties the probe result back to silicon. There are four links:

```
1. Google root attestation CA  (baked into every Android device at factory)
        │
        ▼
2. KeyMint TEE root key  (per-device, sealed in hardware, never leaves the SE)
        │
        ▼
3. Per-attestation leaf key  (freshly generated for THIS probe)
        │   ← attestation challenge bytes = blake2_256(probe_result || timestamp)
        ▼
4. Signature on the probe payload  (signed by the leaf key)
```

Every level is verifiable on chain. The Materios `pallet-tee-attestation` vendors the Acurast verifier and walks the cert chain to the Google root.

If any link is broken — counterfeit device, tampered KeyMint, replaying old attestations, or signing a payload the challenge doesn't commit to — the verifier rejects.

***

## How to Become a Witness

Witness onboarding is intentionally frictionless. You need:

- A modern Android phone with hardware-backed KeyMint (any device shipped after 2018 with API level ≥ 28)
- The Witness APK (download link on https://fluxpointstudios.com/materios/witness)
- An internet connection

### Install + self-register

1. **Scan the QR code at fluxpointstudios.com/materios/witness.** This deep-links the APK and pre-fills the gateway URL.
2. **Open the app, tap "Become a Witness".** The phone:
   - Generates a fresh KeyMint key for the device's witness identity
   - Builds an Android Key Attestation cert chain over it
   - POSTs the chain + a self-generated sr25519 envelope key to `https://materios.fluxpointstudios.com/preprod-blobs/v2/attestor_registration`
3. **Gateway verifies the cert chain** (Google root → device → KeyMint → leaf), creates the on-chain `pallet-tee-attestation::register_attestor` call, and the device is live.

End-to-end install-to-first-probe takes under 60 seconds in practice on a clean phone with no prior witness state.

### What the app does in the background

- Tick interval is configurable on the gateway side (currently 60s).
- Each tick fetches `/witness/targets` and runs every probe in that list.
- For each probe result, the app **generates a fresh KeyMint key** with an attestation challenge equal to `blake2_256(target_url || result || timestamp)`.
- Posts the result + cert chain to the gateway via the `attestation_evidence` route. The gateway accepts polyalg signatures (sr25519, ed25519, secp256r1 KeyMint-native).
- The on-chain event commits the result to chain state.

### Compensation

v1 ships in observe-only mode. v2 (Q3 2026) introduces:

- **MATRA reward emission** per attested probe (Aegis-pool-funded)
- **Slashable bond** for divergent attestations
- **Heat-map dashboard** ranking attestors by uptime + agreement

***

## What Else This Network Can Do

URL uptime is the first probe class. The architecture is general — anything a phone can observe and commit inside KeyMint is a viable Witness Network probe:

| Probe class | Use case | Status |
|---|---|---|
| HTTPS availability | DEX frontend monitoring, infra SLAs | **LIVE** |
| TLS certificate transparency | Detect cert-misissuance against your domain in real time | Q3 2026 |
| Wallet RPC liveness | Independent monitoring of public Cardano + Materios RPCs | Q3 2026 |
| Software version drift | Verify a specific git SHA is what's actually served at a URL | Q4 2026 |
| Geographic latency probes | Edge-network performance baselines | Q4 2026 |
| Compliance auditing (jurisdiction-anchored attest) | Prove a piece of content was unavailable in region X at time T | Exploring |

***

## Network Effects

Every new Witness multiplies the network's value to the others:

- **More witnesses → more URLs covered → better-grained uptime maps.** Adding a phone in Manila doesn't dilute the network — it adds an observation point the others can't replicate.
- **More witnesses → harder to corrupt a result.** An adversary controlling 10% of witnesses can't move the threshold M-of-N reading. The cost of attack scales with N.
- **More witnesses → more MATRA demand.** v2 rewards are funded from the Aegis insurance pool, which is sized by total network capacity.

The network is permissionless. We don't gatekeep on country, carrier, device model, or operator identity. The TEE attestation is the gate.

***

## Architecture Deep Dive

### Pallets

- **`pallet-tee-attestation`** ([materios main, commit 1623de7](https://github.com/Flux-Point-Studios/materios/tree/main/pallets/tee-attestation)) — vendored Acurast verifier; ARM TrustZone path is the only path enabled in v1, AMD SEV-SNP / Intel TDX / build-attestation / ZK paths are stubs gated by Phase 2.5 onboarding.
- **`pallet-orinq-receipts`** (existing) — provides the M-of-N committee + cert chain for the on-chain registry of witnesses. Reused, not forked.
- **`pallet-intent-settlement`** (existing, post-spec-221) — same chain; settlement of attestation-class disputes ride on this.

### Off-chain services

- **Blob Gateway** (TypeScript, `services/blob-gateway`) — `/v2/attestor_registration` for onboarding, `/v2/attestation_evidence` for tick submission, `/witness/targets` for tick-time probe list, polyalg signature verification (sr25519 + ed25519 + secp256r1 KeyMint).
- **WitnessWorker APK** (Kotlin) — Android-side prober. KeyMint key generation, attestation cert chain construction, probe execution, gateway POST.
- **Cert daemon committee** — the same operator-kit committee that attests AvailabilityCert + ClaimSettlement; signs `TeeAttestation::EvidenceVerified` calls.

### Cross-chain

Periodic Materios checkpoints anchor to Cardano L1 via metadata-label-2222. A Witness probe at block #194467 is durably anchored to Cardano via the next Materios → Cardano anchor (typically within 5 minutes).

***

## Roadmap

| Phase | Quarter | Deliverable |
|---|---|---|
| **v1: Live** | Q2 2026 | TEE-attested URL probe, multi-arch verifier, APK self-registration, on-chain anchoring (DONE) |
| **v2: Rewards + slashing** | Q3 2026 | MATRA reward emission, slashable bond, divergent-attestation auto-dispute |
| **v3: Probe diversity** | Q4 2026 | TLS-CT, RPC liveness, version-drift, geographic-latency probes |
| **v4: Subscriber API** | Q1 2027 | Pay-per-query gateway over the witness dataset (consume from any chain via x402 or Cardano metadata) |
| **v5: Federated sub-networks** | Q2 2027 | Domain-scoped witness pools (e.g. "all Manila witnesses", "all Pixel 9+ witnesses") for SLO-shaped queries |

***

## Getting Involved

| You are a… | Next step |
|---|---|
| Android user with a spare phone | https://fluxpointstudios.com/materios/witness — scan + install the APK |
| Cardano project worried about an infra dependency | Open an issue at https://github.com/Flux-Point-Studios/materios proposing a probe class; we'll wire your URL into `/witness/targets` |
| Developer who wants to consume witness data | The gateway's `/witness/observations` API is open + permissionless; pay-per-query lands in v4 |
| Investor / partner | hello@fluxpointstudios.com — the witness fleet is dual-use: same APK ships TEE-attested billing AND TEE-attested distributed QA |

The Witness Network is one of the few places in Web3 where running unbalanced consumer hardware is a load-bearing role, not a tourist task. If you have a phone, you have a job.
