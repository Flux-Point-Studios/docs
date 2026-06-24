---
description: The on-chain shape of Aegis — three Aiken validators, a multi-provider oracle dispatcher, and the five canonical transactions.
---

# How Aegis Works On-Chain

Aegis is a single Aiken project that compiles to three Plutus V3 scripts, a multi-provider oracle
dispatcher (Charli3 / Orcfax / AegisSelf), and five canonical transactions. This is the full shape
of the protocol — datums, redeemers, transaction graphs, and the edge cases they are designed to
handle.

```
aiken/
├─ lib/aegis/
│   ├─ types.ak
│   ├─ oracle.ak
│   ├─ pricing.ak
│   ├─ validation.ak
│   └─ pool.ak
└─ validators/
    ├─ policy.ak
    ├─ pool.ak
    └─ lp_token.ak
```

## 1. Overview — three components, cross-referencing each other

Each script governs a distinct UTxO type, and the three are entangled by design: the pool checks
the policy validator's outputs, the LP minting policy is parameterized by the pool's script hash,
and policies route their residual back to the pool when they expire or claim.

```
   Policy Validator        Pool Validator         LP Token Policy
   governs each            shared liquidity        parameterized by
   policy UTxO     ──────► + reserves    ◄──────   the pool hash
   (expire/claim residual)  (underwrite reserve)   (require pool input)
```

**Why three scripts and not one?** Separation of concerns. The policy validator is per-policy and
short-lived; the pool validator is a single shared UTxO with a long lifetime; the LP minting
policy is stateless and only authorizes mint/burn on consumption of the pool. Splitting them keeps
each script small, auditable, and independently upgradeable.

## 2. Policy validator — one UTxO per policy, three redeemer paths

Each policy is a single UTxO at the policy validator's address. Its inline datum encodes everything
the on-chain logic needs to settle the contract — no metadata server required.

```aiken
type PolicyDatum {
  policy_id:        ByteArray,                  // hash of terms
  insured:          VerificationKeyHash,        // owner
  strike_price:     Int,                        // ADA/USD ≤ this triggers payout (×1e6)
  coverage_amount:  Int,                        // payout cap, lovelace
  premium_paid:     Int,                        // premium escrowed, lovelace
  start_time:       Int,                        // POSIX ms — earliest valid claim
  expiry_time:      Int,                        // POSIX ms — latest valid claim
  oracle_nft:       ByteArray,                  // oracle feed identity (Charli3-compatible)
  oracle_provider:  Int,                        // 0 = Charli3, 1 = Orcfax, 2 = AegisSelf
  pool_address:     ByteArray,                  // where residual returns
}
```

```aiken
type PolicyRedeemer {
  Claim    // oracle ≤ strike, in window        → pays insured + returns residual
  Expire   // tx lower bound > expiry_time       → returns full UTxO to pool
  Cancel   // signed by insured, within grace    → premium refund minus fee
}
```

**Claim path — what gets verified:**

1. A reference input contains the oracle UTxO (matched by `oracle_nft`) for the configured
   `oracle_provider` — Charli3, Orcfax, or AegisSelf.
2. The oracle datum parses as `OracleDatum` and is not expired (`oracle_expiry > tx.lower_bound`).
3. Oracle `price ≤ strike_price`.
4. The validity range is within `[start_time, expiry_time]`.
5. An output pays `≥ coverage_amount` to `insured`'s payment credential.
6. The remainder of the policy UTxO routes to `pool_address`.

**Cancel path:** signed by the insured, within the grace window after `start_time`, refunds the
premium minus the cancellation fee, remainder to pool. The grace window protects underwriters from
policies bought during a flash spike then immediately cancelled.

> Protocol constants live in `lib/aegis/types.ak` — `min_premium = 2_000_000`,
> `max_coverage_ratio = 50`, `price_scale = 1_000_000`, plus the cancellation window and fee.

## 3. Pool validator — one shared UTxO, four redeemer paths

A single canonical UTxO holds all underwriter capital. Every operation consumes it and recreates
it with updated datum. The pool is identified by an NFT — there can only be one, so concurrency is
enforced at the protocol level.

```aiken
type PoolDatum {
  total_liquidity:  Int,        // ADA deposited by LPs (lovelace)
  active_coverage:  Int,        // ADA reserved against open policies (lovelace)
  lp_token_policy:  ByteArray,  // minting policy ID for aLP
  protocol_fee_bps: Int,        // 200 = 2%
  pool_nft:         ByteArray,  // identifies THIS pool UTxO
}

type PoolRedeemer {
  Underwrite      { coverage: Int, premium: Int }   // reserve coverage for a new policy
  ProcessClaim    { payout: Int }                   // settle a claim
  AddLiquidity    { amount: Int }                   // mint aLP, increase liquidity
  RemoveLiquidity { amount: Int }                   // burn aLP, withdraw share
}
```

**The solvency invariant**, enforced across all four redeemers:

```
total_liquidity ≥ active_coverage      (always)
available = total_liquidity - active_coverage

Underwrite:      coverage ≤ available  AND  coverage / premium ≤ 50
RemoveLiquidity: amount   ≤ available  (cannot withdraw reserved funds)
```

**LP math** — standard CPMM-style shares; integer truncation is a tiny rounding residual that
always favors the pool:

```
mint     = (deposit   × lp_supply)       / total_liquidity   // 1:1 if pool empty
withdraw = (lp_burned × total_liquidity) / lp_supply
```

## 4. LP minting policy — a stateless authorizer parameterized by the pool

The `aLP` minting policy is parameterized at compile time by the pool validator's script hash. Its
only job is to refuse mint or burn unless the pool is being consumed in the same transaction.

```aiken
validator lp_token(pool_script_hash: ByteArray) {
  mint(redeemer: LPTokenRedeemer, _policy_id: PolicyId, self: Transaction) {
    // 1. The pool validator MUST be in the inputs of this transaction.
    let pool_consumed =
      list.any(self.inputs, fn(i) {
        when i.output.address.payment_credential is {
          Script(h) -> h == pool_script_hash
          _         -> False
        }
      })
    // 2. Mint amount sign matches the redeemer.
    let mint_qty = assets.quantity_of(self.mint, self.policy_id, "aLP")
    when redeemer is {
      MintLP -> pool_consumed && mint_qty > 0
      BurnLP -> pool_consumed && mint_qty < 0
    }
  }
}
```

**Why parameterize?** The pool's script hash is baked into the LP policy ID, so the asset class is
uniquely bound to one pool. You cannot mint `aLP` without consuming the authoritative pool UTxO —
the pool's own checks (deposit value, datum delta) become the mint authorization by transitive
dependency.

## 5. Oracle integration — multi-provider dispatch via CIP-31 reference inputs

The oracle UTxO is never consumed. Aegis reads it as a **reference input** (CIP-31), so one feed
can support hundreds of concurrent claims per epoch. The datum is the Charli3-compatible
`GenericData` shape — Aegis only verifies, never produces.

```aiken
type OracleProvider {
  Charli3    // 0 — Charli3 ODV pull oracle, GenericData datum
  Orcfax     // 1 — Orcfax FSP feed
  AegisSelf  // 2 — Aegis-published median (self-publish)
}

type PriceData {
  SharedData
  ExtendedData
  GenericData { price_map: Pairs<Int, Int> }
}
type OracleDatum { price_data: PriceData }

// Helper extractors
fn get_oracle_price    (d: OracleDatum) -> Int     // map[0] = price (×1e6)
fn get_oracle_timestamp(d: OracleDatum) -> Int     // map[1] = creation
fn get_oracle_expiry   (d: OracleDatum) -> Int     // map[2] = expiry
fn is_oracle_valid     (d, now)         -> Bool    // expiry > now
fn find_oracle_input(refs, oracle_nft)  -> Input   // by NFT policy id
```

Each policy datum carries an `oracle_provider` tag and an `oracle_nft`; the on-chain dispatcher
picks the matching reference-input verifier at claim time, so the same protocol settles policies
against any supported provider without redeploying a validator. New providers plug in by adding a
variant to the sum type and mapping it to a verifier.

**Why reference, not consume?**

1. **Concurrency.** Each provider publishes one UTxO per feed per block. Consuming it would limit
   settlement to one claim per block; reference inputs let arbitrarily many transactions read it.
2. **Freshness.** The oracle's `expiry` is enforced inside the validator
   (`oracle_expiry > tx.lower_bound`). Stale feeds are rejected on-chain.
3. **Trust-minimization.** Charli3 is a multi-node aggregator with on-chain median calculation;
   Orcfax FSP and the AegisSelf publisher feed offer redundant paths. Aegis does no off-chain price
   work — it only verifies the reference UTxO at claim time.

## 6. The five canonical transactions

Every off-chain action compiles down to one of these five shapes.

**Buy Policy** — `RDM Pool: Underwrite { coverage, premium }` · `SIG user`
```
INPUTS                          OUTPUTS
[0] Wallet (premium+fee+coll) → [0] Policy UTxO @ policy_validator (value: coverage · PolicyDatum)
[1] Pool UTxO (consumed)         [1] Pool UTxO @ pool_validator (value += premium · +coverage,+premium)
                                 [2] Change → user wallet
```

**Claim Policy** — `RDM Policy: Claim` · ref: oracle UTxO · validity `oracle_expiry > now`
```
INPUTS                 REFERENCE INPUTS
[0] Policy UTxO   ref: Oracle UTxO (Charli3 / Orcfax / AegisSelf), by oracle_nft (not spent)
OUTPUTS
[0] Payout to insured (coverage_amount lovelace)
[1] Residual to pool (policy_value − coverage_amount)
```
The pool is not consumed here — only the policy. Residual returns to `pool_address` as a normal
payment, no pool datum delta needed.

**Expire Policy** — `RDM Policy: Expire` · validity `lower_bound > expiry_time`
```
[0] Policy UTxO  →  [0] Full value to pool_address (no datum update — pure return)
```

**Add Liquidity** — `RDM Pool: AddLiquidity { amount }` + `LPToken: MintLP` · mint `+aLP`
```
[0] LP wallet (deposit) → [0] Pool UTxO (value += deposit · total_liquidity += amount)
[1] Pool UTxO            [1] aLP tokens to LP (amount × lp_supply / total_liquidity)
                         [2] Change
```

**Remove Liquidity** — `RDM Pool: RemoveLiquidity { amount }` + `LPToken: BurnLP` · mint `−aLP`
```
[0] LP wallet (aLP)  → [0] Pool UTxO (value −= withdrawal · total_liquidity −= amount)
[1] Pool UTxO          [1] Withdrawal to LP (lp_burned × total_liquidity / lp_supply)
                       [2] Change
```
Withdrawal is gated by available liquidity: `amount ≤ total_liquidity − active_coverage`.

## 7. Edge cases

- **Oracle unavailable.** If the provider's reference UTxO can't be located, the claim transaction
  simply fails to balance — the validator won't accept a substitute. The off-chain keeper retries
  on the next polling cycle once the provider publishes a new UTxO. Multi-oracle support lets
  operators run parallel policies against different providers for redundancy.
- **Pool fully utilized.** `active_coverage ≥ total_liquidity` blocks new policies at the validator
  level. The off-chain service pre-checks available liquidity and surfaces a clear error rather
  than letting a user sign a doomed transaction.
- **Concurrent claims.** Each policy UTxO is independent, so claims settle in parallel. The pool
  UTxO can only be consumed once per block, but the simple per-policy claim (which doesn't update
  pool datum) sidesteps that bottleneck entirely.
- **Rounding.** Aiken integer arithmetic truncates down, so the pool accrues a few lovelace of dust
  over time, compounding for existing LPs.
- **Minimum UTxO ADA.** Every Cardano UTxO must hold ~1.6 ADA; policy UTxOs always carry enough to
  cover this, and the residual return on expiry/claim is reduced by exactly that amount.
