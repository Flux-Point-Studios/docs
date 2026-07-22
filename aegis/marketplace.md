---
description: Aegis as a risk-infrastructure layer — the participants on every side of the market and how each strengthens the network for the others.
---

# The Seven-Sided Marketplace

Aegis isn't just insurance — it's a risk-infrastructure layer. Several distinct participants each
strengthen the network for every other. This page walks each side: what they pay, what they earn,
how value flows, and where they're exposed.

## 1. Underwriters — liquidity providers

Capital providers who deposit ADA into the shared pool and back every policy the protocol writes,
pro-rata to their `aLP` share. Yield is real economic activity — premiums on policies that expire
without a claim — not token emissions.

- **Pays in:** ADA into the pool; capital is locked while utilized.
- **Earns:** premium yield pro-rata to `aLP` share; scales with utilization.
- **Flows:** deposit ADA → pool (single-UTxO continuation) · mint transferable `aLP` · earn
  premiums on every unclaimed policy · pool drawn pro-rata to pay valid claims.
- **Risks:** drawdown if many policies hit strike at once; `aLP` redemption is gated when
  utilization spikes (to protect open policies); premium yield correlates with market fear — high
  APY often precedes high losses.

### V8 improvements for underwriters

- **Non-custodial deposits/withdrawals** — you sign every transaction. No operator key ever
  touches your funds.
- **Registry governance** — M-of-N admin (not single operator) with 72h timelock on logic changes.
  Admin can propose parameter updates but never has LP custody.
- **Migration path** — V7 LPs can withdraw from the old pool and deposit into the vault without
  losing position.

## 2. Borrowers — liquidation-protected

CDP holders on Indigo, Liqwid, Danogo, Surf, and FluidTokens who attach a policy calibrated to
their liquidation price. If the market crashes, the payout can settle before the liquidation bots
strike — giving the borrower time to top up collateral.

- **Pays in:** a premium tuned to LTV and strike distance.
- **Gets:** a pre-liquidation payout and time to recover the position.
- **Flows:** attach premium → policy (datum links to the CDP) · off-chain keeper tracks LTV vs.
  oracle · payout settles before the keeper liquidation tx wins.
- **Risks:** the strike must sit above the liquidation threshold to be useful; a race against
  keeper bots during cascades; cover ends at expiry — chronic under-collateralization isn't
  insurable.

## 3. Speculators — synthetic-put buyers

The contract doesn't check whether you hold a loan — anyone can buy protection at any strike. That
makes Aegis a pure price-contingent instrument: pay a small premium, receive a large payout if the
price falls below your strike. Sophisticated traders use it as a synthetic put on ADA.

- **Pays in:** premium; no collateral, no margin call.
- **Gets:** an asymmetric payoff — bounded loss, leveraged crash exposure.
- **Flows:** open (premium → policy NFT minted to wallet) · hold (the policy NFT is transferable) ·
  settle (on strike: full coverage paid; else the premium expires).
- **Risks:** premium is non-refundable if the strike never hits in the window; strike-distance
  pricing is actuarial, not arbitrage-free; deep-OTM liquidity is thin until the pool grows.

## 4. Lending protocols — B2B integrators

Indigo, Liqwid, Danogo, Surf, FluidTokens — protocols that embed Aegis into their CDP creation flow
("open a CDP with built-in liquidation insurance"). They reduce systemic liquidation cascades,
protect users, and differentiate the product. Premiums can be subsidized from protocol treasuries.

- **Pays in:** integration time; CIP-31 reference inputs in the CDP datum.
- **Gets:** a lower bad-debt rate; revenue share on referred premiums.
- **Flows:** the Aegis policy NFT is cited in the CDP datum at open time · insolvency on covered
  positions falls during cascades · a share of premium on protocol-referred policies.
- **Risks:** tight coupling (an Aegis pause cascades to integrator UX); a mis-calibrated subsidy
  could create moral hazard; reference-input contention during high-throughput epochs.

## 5. Treasuries & DAOs — risk managers

Funded projects, DAOs, and protocol treasuries holding significant ADA can buy crash protection as
basic risk management. A 500K-ADA treasury buying 30-day cover at a 20%-below strike costs roughly
1–2% — trivial insurance against catastrophic loss.

- **Pays in:** a premium budget, roughly 1–2% of treasury per cycle.
- **Gets:** a floor on treasury value, fiduciary cover, runway certainty.
- **Flows:** allocate (multi-sig premium tx via governance vote) · hold (policy NFT custodied at
  the treasury address) · disclose (on-chain hedge ratio visible to the community).
- **Risks:** premium is an explicit P&L line to defend at the next vote; per-policy coverage caps
  mean large treasuries split across policies; strike calibration is a governance question.

## 6. Arbitrageurs — pricing efficiency

When Aegis premiums diverge from CEX-implied volatility, arbitrageurs buy the cheaper instrument
and sell the dearer. This compresses the spread and keeps Aegis pricing reflecting real market
risk — they are unpaid market makers.

- **Pays in:** capital, monitoring infrastructure, execution latency.
- **Gets:** the spread between the Aegis premium and the CEX-IV-implied option price.
- **Flows:** watch (Aegis premium vs. CEX option mid) · trade (buy the under-priced side, hedge on
  the other venue) · pricing converges to fair IV, protecting pool capital.
- **Risks:** cross-venue settlement / slippage / withdrawal latency; counterparty risk on the CEX
  leg; an on-chain Aegis policy can't be cancelled while the CEX leg can — basis blow-out possible.

## 7. The broader market — an on-chain fear index

Aggregate Aegis demand at every strike and duration becomes Cardano's first on-chain fear gauge —
analogous to the VIX. When demand spikes at a low strike, the market is pricing in a crash. The
signal is on-chain, real-time, and consumable by any protocol via reference inputs.

- **Pays in:** nothing — it's derived from existing protocol activity.
- **Gets:** a public good: a transparent volatility / fear primitive.
- **Flows:** aggregate (strike concentration · premium velocity · utilization) · publish (an
  AEGIS / FEAR datum each block) · other protocols consume it via CIP-31 reference inputs.
- **Risks:** thin volume makes the index noisy (robust only above a baseline TVL); manipulable in
  early epochs by a single large policy at an extreme strike; the index definition is open and
  could fork as new feeds arrive.

## 8. The Cardano treasury — a validator-enforced give-back

Every fee-bearing Aegis transaction routes a fixed share of the protocol fee directly into the
on-chain Cardano protocol treasury via the Conway-era `treasury_donation` body field (CDDL key 22).
The pool validator **rejects** any fee-bearing transaction that under-pays — it's enforced
cryptographically, not by trust. With the default parameters (a 2% protocol fee × a 25% treasury
share), **0.50% of every premium** flows to Cardano governance reserves at the next epoch
transition.

- **Pays in:** 0.50% of every Aegis premium (25% of the protocol fee).
- **Gets:** sustainable funding for Cardano governance, Catalyst, and dev tooling.
- **Flows:** the validator derives the required cut on-chain (no off-chain trust) · body field 22
  carries the lovelace into the treasury at the epoch boundary · every donating tx is visible on
  cardanoscan with a "Treasury Donation" row.
- **Risks:** hardware-wallet support for Conway donations isn't yet universal (feature-flagged
  behind a CIP-30 capability check); the validator hash rotates if the share constant changes, so
  the rate is code-pinned rather than operator-mutable — by design.
