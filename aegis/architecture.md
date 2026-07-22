---
description: The on-chain shape of Aegis — from V4 pool to V8 stable-vault. Four Aiken validators, a marker-based integrity layer, a multi-provider oracle dispatcher, pool-funded coverage with a validator-enforced treasury donation, and the canonical transactions.
---

# How Aegis Works On-Chain

Aegis is a single Aiken project that compiles to **four runtime Plutus V3 scripts** plus a one-shot
NFT that bootstraps the pool. This page covers both the **V4 protocol** (historical) and the
currently deployed **V8 stable-vault** — datums, redeemers, the marker integrity layer, the oracle
dispatcher, the pool-funded pricing model, and the transaction graphs the validators interlock to
enforce.

```
aiken/
├─ lib/aegis/                      reusable logic
│   ├─ types.ak     ├─ pricing.ak  ├─ oracle.ak / oracle/*.ak
│   └─ pool.ak      └─ validation.ak
└─ validators/
    ├─ policy.ak          # spend  — the policy validator
    ├─ pool.ak            # spend  — the pool validator (V4)
    ├─ pool_vault.ak      # spend  — the vault validator (V8)
    ├─ lp_token.ak        # mint   — the aLP minting policy
    ├─ policy_marker.ak   # mint   — the marker minting policy (integrity)
    └─ pool_nft.ak        # mint   — one-shot pool NFT (genesis only)
```

## 1. Overview — four scripts, entangled by design

| Script | Governs | Lifetime |
|--------|---------|----------|
| **Policy validator** | one UTxO per policy | short-lived |
| **Pool validator** | the single shared liquidity UTxO | long-lived (V4) |
| **Vault validator (V8)** | the single shared vault UTxO | long-lived (V8) |
| **aLP minting policy** | `aLP` LP tokens, parameterized by the pool hash | stateless |
| **Marker minting policy** | per-policy `AEGIS_POLICY` marker tokens | stateless |
| _pool NFT (one-shot)_ | identifies the canonical pool | minted once at genesis |

The scripts cross-reference each other: the **pool/vault funds and bears the coverage** for every
policy it underwrites; the **aLP policy** can only mint/burn when the pool is consumed in the same
tx; the **marker policy** can only mint/burn when the canonical pool is consumed; and each
**policy** routes its residual back to the pool when it claims, cancels, or expires. Splitting
concerns keeps every script small, auditable, and independently upgradeable, while the markers
stitch the lifecycle together so it can't be forged.

### V8 Stable-Vault

V8 replaces the V7 `PoolDatum` (6–7 fields) with a leaner `VaultState` (4 fields) and introduces
a **registry** for governance:

- `lp_supply` — outstanding LP tokens
- `active_coverage` — capital reserved against open policies
- `reserve` — pool reserve
- `registry_nft` — ties the vault to a governance registry

The vault derives `total_liquidity = backing + active_coverage` (where `backing = own − min_utxo −
reserve`) rather than storing it. The frontend consumes the same 6-key JSON contract, so no UI
changes are required.

## 2. Policy validator — one UTxO per policy

Each policy is a UTxO at the policy validator's address. Its inline datum encodes everything needed
to settle the contract, plus the bindings that prevent forgery.

### V4 PolicyDatum (14 fields)

```aiken
type PolicyDatum {
  policy_id:         ByteArray,            // hash of terms
  insured:           VerificationKeyHash,  // owner / payout recipient
  strike_price:      Int,                  // trigger price (×1e6)
  coverage_amount:   Int,                  // payout, lovelace (pool-funded)
  premium_paid:      Int,                  // premium, lovelace
  start_time:        Int,                  // POSIX ms
  expiry_time:       Int,                  // POSIX ms
  oracle_nft:        ByteArray,            // feed identity
  pool_script_hash:  ScriptHash,           // pins THIS pool
  pool_nft:          ByteArray,            // pins THIS pool UTxO
  oracle_provider:   OracleProvider,       // Charli3 | Orcfax | AegisSelf | Indigo
  partner_address:   Option<Address>,      // optional referral fee recipient
  partner_share_bps: Int,                  // ≤ partner_share_cap_bps (2000)
  risk_class:        RiskClass,            // Barrier | Depeg
}
```

### V8 PolicyDatum (17 fields)

V8 expands the datum to 17 fields, adding observer attestation and eligibility logic bindings:

```aiken
type PolicyDatum {
  // ... fields 1-14 same as V4 ...
  observer_nft:      ByteArray,            // pinned observer feed
  eligibility_logic: ScriptHash,           // eligibility validator hash
  valid_until:       Int,                  // observer reading expiry
}
```

The additional fields support the **observer attestation** pattern: the validator resolves the
oracle feed, checks freshness against a `valid_until` ceiling (the K-1 constraint), and verifies
the eligibility redeemer matches the policy's `eligibility_logic` hash.

Every policy UTxO **must carry exactly one marker token** `(policy_id, "AEGIS_POLICY")` minted by
the marker policy (see §5). `Cancel`, `Claim`, and `Expire` only spend a policy that holds its
marker — so a hand-written "policy" UTxO that was never underwritten can never drain the pool.

- **Claim** — a reference input holds the oracle UTxO (matched by `oracle_nft` for the policy's
  `oracle_provider`); the reading is fresh and meets the strike for the policy's `risk_class`; the
  validity range is within `[start_time, expiry_time]`; the insured is paid `coverage_amount`; the
  paired `ProcessClaim` on the pool/vault burns the marker and updates accounting.
- **Expire** — `tx.lower_bound > expiry_time`; the residual returns to the pool; the paired
  `BatchExpireProcess` burns the marker.
- **Cancel** — signed by the insured within the cancellation window (`3_600_000 ms`); refund =
  `premium_paid − 10%` (`cancellation_fee_bps = 1000`); the paired `AcceptCancellation` on the pool
  burns the marker.
- **BatchClaim / BatchExpire** — the same logic, vectorized over many policies in one transaction.

> Constants (`lib/aegis/types.ak`): `max_coverage_ratio = 50`, `cancellation_window = 3_600_000`,
> `cancellation_fee_bps = 1_000`, `price_scale = 1_000_000`, `min_utxo_lovelace = 2_000_000`,
> `partner_share_cap_bps = 2_000`, marker asset name `"AEGIS_POLICY"`.

## 3. Pool validator — one shared UTxO, seven redeemer paths (V4)

A single canonical UTxO (identified by the pool NFT) holds all underwriter capital. Every operation
consumes it and recreates it with updated datum.

```aiken
type PoolDatum {
  total_liquidity:  Int,        // ADA deposited by LPs (lovelace)
  active_coverage:  Int,        // reserved against open policies (lovelace)
  lp_token_policy:  ByteArray,  // the aLP minting policy id
  protocol_fee_bps: Int,        // 200 = 2%
  pool_nft:         ByteArray,  // identifies THIS pool UTxO
  lp_supply:        Int,        // outstanding aLP
}

type PoolRedeemer {
  Underwrite        { coverage: Int, premium: Int }
  ProcessClaim      { payout: Int }
  AddLiquidity      { amount: Int }
  RemoveLiquidity   { amount: Int }
  BatchUnderwrite   { total_coverage: Int, total_premium: Int }
  BatchExpireProcess{ total_returned: Int }
  AcceptCancellation
}
```

**Pool-funded coverage.** Unlike a model where the buyer escrows the payout, the **pool funds the
coverage** and the buyer pays only the premium. On `Underwrite` the pool value moves by:

```
fee_total       = max(min_utxo, premium × protocol_fee_bps / 10_000)   // floor inside the carve
net_pool_growth = premium − fee_total
pool_out_value  = pool_in_value + net_pool_growth − coverage           // value_ok
datum:  total_liquidity += net_pool_growth ,  active_coverage += coverage ,  lp_supply unchanged
```

**Solvency + concentration**, enforced across the redeemers:

```
available = total_liquidity − active_coverage
Underwrite:       coverage ≤ available  AND  coverage / premium ≤ 50
accumulation cap: active_coverage_after × 3 ≤ total_liquidity_after   // ≤ ⅓ of pool capital
RemoveLiquidity:  amount ≤ available                                  // can't withdraw reserved funds
```

### V8 Vault validator

The V8 vault replaces `PoolDatum` with `VaultState` and adds a **registry**:

```aiken
type VaultState {
  lp_supply:        Int,        // outstanding aLP
  active_coverage:  Int,        // reserved against open policies
  reserve:          Int,        // pool reserve
  registry_nft:     ByteArray,  // ties to governance registry
}

type VaultRedeemer {
  AddLiquidity
  RemoveLiquidity
  Underwrite        { coverage: Int, premium: Int }
  Claim             { coverage: Int }
  Cancel
  Expire
}
```

The vault is **non-custodial**: the connected user funds and receives. The operator key never
touches user funds. Six lifecycle paths all route through the vault with the user signing their own
transaction.

**Registry governance.** The vault is tied to a `RegistryDatum` with M-of-N admin (configurable at
genesis via `--admin-vkeys` and `--admin-threshold`). The admin can only propose logic swaps behind
a 72h timelock — it never has LP custody.

**Multi-feed mandate.** V8 supports multiple oracle feeds via `allowed_oracle_nfts` (a list, not a
single feed). A `--deny-all-underwrites` flag sets an empty mandate, closing all underwrites while
leaving LP add/remove intact.

**LP math** — identical to V4, but `total_liquidity` is derived:

```
backing = own_value − min_utxo − reserve
total_liquidity = backing + active_coverage
mint     = (deposit   × lp_supply) / total_liquidity
withdraw = (lp_burned × total_liquidity) / lp_supply
```

## 4. aLP minting policy — a stateless authorizer parameterized by the pool

The `aLP` policy is parameterized at compile time by the pool validator's script hash. It refuses
to mint or burn unless the pool is consumed in the same transaction, so the pool's own checks
(deposit value, datum delta) become the mint authorization by transitive dependency. The pool's
script hash is baked into the `aLP` policy id, binding the asset class to exactly one pool.

```aiken
validator lp_token(pool_script_hash: ByteArray) {
  mint(redeemer: LPTokenRedeemer, _pid: PolicyId, self: Transaction) {
    let pool_consumed = list.any(self.inputs, fn(i) {
      when i.output.address.payment_credential is { Script(h) -> h == pool_script_hash; _ -> False } })
    let qty = assets.quantity_of(self.mint, self.policy_id, "aLP")
    when redeemer is { MintLP -> pool_consumed && qty > 0 ; BurnLP -> pool_consumed && qty < 0 }
  }
}
```

## 5. Marker minting policy — the integrity layer (new in V4)

The marker policy is the piece that makes the lifecycle un-forgeable. **Every policy UTxO must hold
exactly one `(policy_id, "AEGIS_POLICY")` marker.** The pool mints markers only on `Underwrite` /
`BatchUnderwrite`, and burns them only on `ProcessClaim` / `AcceptCancellation` /
`BatchExpireProcess`.

```aiken
type MarkerRedeemer {
  MintMarkers { count: Int }     // 1 marker per new policy output
  BurnForClaim                   // paired with ProcessClaim
  BurnForCancel                  // paired with AcceptCancellation
  BurnForExpire { count: Int }   // paired with BatchExpireProcess
}
```

**Why it exists.** Without it, an attacker could write a *fake* policy UTxO with an inflated
`premium_paid`, fund it with 2 ADA, and call `Cancel` (or `Claim`) to drain real lovelace from the
pool (the original EXT-08 / EXT-09 critical findings). A marker can only be minted by a real
`Underwrite` — which co-spends the canonical pool — so a policy without a valid marker simply can't
be spent on the refund/payout paths.

**Mint validation** (`MintMarkers { count }`): the canonical pool is consumed (an input carries the
one-shot pool NFT — pinning the *specific* pool, not just the script address); `count > 0`; `count`
equals the number of new policy outputs at the policy validator; the total minted under
`(own_policy, "AEGIS_POLICY")` equals `count`; and every new policy output holds exactly one marker.

**Burn validation:** the canonical pool is consumed (so only the pool's burn-authorizing redeemers
can co-spend a marked policy), the net marker quantity is negative, and no positive entries piggy-
back under another asset name. The marker policy is parameterized by the pool-NFT policy id + asset
name + the policy validator hash, in a strict no-cycle DAG (`pool_nft → policy_marker → policy → pool → lp_token`).

## 6. Oracle integration — multi-provider dispatch via CIP-31 reference inputs

The oracle UTxO is never consumed; Aegis reads it as a **reference input** (CIP-31), so one feed
backs many concurrent claims. The policy datum names an `oracle_provider` and an `oracle_nft`, and
the on-chain dispatcher routes to the matching provider's resolver:

```aiken
type OracleProvider { Charli3  Orcfax  AegisSelf  Indigo }
```

- **Charli3** — multi-node ODV pull oracle, on-chain median, Charli3-compatible `GenericData` datum.
- **Orcfax** — FSP feed (`CER/ADA-USD/`), pinned feed-id + asset name, 70-minute freshness window.
- **AegisSelf** — Aegis's own published median. The resolver pins the feed two ways: the
  `oracle_nft` must be in the canonical AegisSelf allowlist **and** the feed UTxO must sit at the
  publisher's verification-key hash. A forged NFT under an attacker policy is rejected before the
  payment-credential check even runs.
- **Indigo** — per-iAsset oracle, NFT canonical-membership + oracle-script-credential pin.

Across providers the validator enforces freshness against the tx validity range (a stale feed is
rejected on-chain) and only verifies — Aegis does no off-chain price-oracle work of its own.

### V8 observer attestation

V8 adds an **observer attestation** pattern for oracle-bearing transactions:

- The oracle feed is resolved into 5 `Price` fields (spot, 12h low, etc.)
- An **observer script** attests to the feed's integrity (byte-match check)
- The **eligibility logic** validator verifies the insured's eligibility via a withdrawal sibling
- Freshness is two-legged: `tx_lower ≤ observed_at + 5 min` and `tx_upper ≤ valid_until`

The 4-redeemer / 2-withdrawal shape (Underwrite) ensures every oracle-bearing path carries both
the observer attestation and the eligibility check.

## 7. The canonical transactions

### V4/V7 — Pool-based

**Buy coverage (`Underwrite`)** — the pool funds the coverage; the buyer pays the premium.
```
INPUTS                                   OUTPUTS / MINT
[ ] Wallet (premium + fees + collateral) [ ] Policy UTxO @ policy_validator
[ ] Pool UTxO (consumed)                     value = coverage + 1 marker · 14-field PolicyDatum
ref: Oracle UTxO (for the floor read)    [ ] Pool UTxO (value += net_growth − coverage; datum +cov,+growth)
                                         [ ] Team fee output  ( [ ] Partner fee output, if any )
                                         [ ] Change → wallet
                                         MINT  +1 (policy_id, "AEGIS_POLICY")
                                         BODY  treasury_donation (field 22) ≥ required cut
RDM  Pool: Underwrite{coverage,premium} · Marker: MintMarkers{1}
```

**Claim** — pay the insured; the pool's `ProcessClaim` burns the marker. (`BatchClaim` vectorizes.)
```
[ ] Policy UTxO (marked, consumed)   ref: Oracle UTxO (price ≤ strike, fresh)
[ ] Pool UTxO (consumed)             →  [ ] Payout to insured (coverage_amount)
                                        [ ] Pool UTxO (active_coverage −= coverage)
                                        MINT −1 marker      RDM Policy: Claim · Pool: ProcessClaim · Marker: BurnForClaim
```

**Cancel** — refund the premium minus the fee within the grace window; marker burned via
`AcceptCancellation`. **Expire** (`BatchExpire`) — after `expiry_time`, the residual returns to the
pool and the marker is burned via `BatchExpireProcess`.

**Add / Remove liquidity** — mint/burn `aLP` while consuming the pool:
```
Add:    [Wallet deposit][Pool] → [Pool (+deposit, total_liquidity += amount)][aLP to LP][Change]   MINT +aLP
Remove: [Wallet aLP   ][Pool] → [Pool (−withdrawal, total_liquidity −= amount)][ADA to LP][Change] MINT −aLP
        // withdrawal gated by available = total_liquidity − active_coverage
```

### V8 — Vault-based (non-custodial)

**Buy coverage (`Underwrite`)** — identical economics, but non-custodial:
```
INPUTS                                   OUTPUTS / MINT
[ ] Wallet (premium + fees + collateral) [ ] Policy UTxO @ policy_validator
[ ] Vault UTxO (consumed)                    value = coverage + 1 marker · 17-field PolicyDatum
ref: Oracle UTxO (for the floor read)    [ ] Vault UTxO (value += premium − coverage; active_coverage += cov)
                                         [ ] Team fee output
                                         [ ] Change → wallet
                                         MINT  +1 (policy_id, "AEGIS_POLICY")
                                         BODY  treasury_donation (field 22) ≥ required cut
                                         WITHDRAW  observer attestation + eligibility_logic
RDM  Vault: Underwrite{coverage,premium} · Marker: MintMarkers{1}
```

**Claim / Cancel / Expire** — identical to V4, but against the vault UTxO and using the V8
`BurnForClaim` / `BurnForCancel` / `BurnForExpire` marker redeemers.

**Add / Remove liquidity** — identical math, but the vault derives `total_liquidity` from `backing +
active_coverage`:
```
Add:    [Wallet deposit][Vault] → [Vault (+deposit, backing += amount)][aLP to LP][Change]   MINT +aLP
Remove: [Wallet aLP   ][Vault] → [Vault (−withdrawal, backing −= amount)][ADA to LP][Change] MINT −aLP
```

## 8. Edge cases & invariants

- **Synthetic-policy drain — closed.** A policy can't be claimed or cancelled without a marker, and
  only a real `Underwrite` mints one (§5).
- **Finite validity required.** The floor and claim paths read the tx validity range as POSIX ms
  and `fail` on a non-finite bound, so every Underwrite/Claim must set both `invalid_before` and
  `invalid_hereafter`.
- **Lifecycle pairing.** Marker mints/burns are paired with specific pool redeemers
  (`MintMarkers↔Underwrite`, `BurnForClaim↔ProcessClaim`, `BurnForCancel↔AcceptCancellation`,
  `BurnForExpire↔BatchExpireProcess`), so a policy can't be settled on the wrong path.
- **Concentration cap.** `active_coverage` can never exceed ⅓ of `total_liquidity`, so one
  correlated crash leaves the pool solvent with the majority of its capital intact.
- **Solvency.** `total_liquidity ≥ active_coverage` always; LPs can't withdraw reserved funds.
- **Rounding & min-UTxO.** Integer truncation accrues dust to the pool (favoring LPs); every policy
  UTxO carries ≥ `min_utxo_lovelace` (2 ADA), and residual returns are reduced by exactly that.
- **V8 vault non-custody.** The operator address never appears in user-funded transactions. The
  user funds, signs, and receives all outputs.
- **V8 registry timelock.** Admin proposals require 72h before execution; admin can never withdraw
  LP funds.
