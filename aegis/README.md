---
description: Aegis — parametric, on-chain insurance and risk infrastructure for Cardano
---

# Aegis

Aegis is **parametric insurance built natively on Cardano**. Buyers escrow a small premium and
receive an automatic, oracle-settled payout if a price condition is met — an ADA crash below a
chosen strike, a stablecoin depeg, or a CDP nearing liquidation. Underwriters provide pooled ADA
liquidity that backs every policy and earn the premiums on policies that expire without a claim.
There is no claims desk and no custodian: settlement is enforced by Aiken validators, and the
oracle reading is verified on-chain at claim time.

Live on Cardano mainnet at [aegis.fluxpointstudios.com](https://aegis.fluxpointstudios.com).

## What you can do with Aegis

- **Buy crash coverage** — a price-contingent payout if ADA (or iBTC / iETH) falls below your
  strike before expiry. Functionally a synthetic put: bounded premium, leveraged downside payout.
- **Buy depeg coverage** — protection on stablecoins (USDC, USDT, iUSD) that pays if the feed
  leaves its insurable band around the peg.
- **Protect a CDP** — attach a policy calibrated to your liquidation price on Indigo, Liqwid,
  Danogo, Surf, or FluidTokens, so a payout can settle before the liquidation bots strike.
- **Underwrite** — deposit ADA into the shared pool, receive transferable `aLP` tokens, and earn
  premium yield pro-rata to your share.

## Design goals

- **Non-custodial by construction.** Every action is a Cardano transaction the user signs in their
  own wallet. Aegis never holds keys or funds.
- **Parametric, not discretionary.** Payouts are decided by an on-chain price condition and an
  oracle reading the validator checks itself — not by a human adjudicator.
- **Solvency enforced on-chain.** The pool validator never lets reserved coverage exceed deposited
  liquidity, and never lets an underwriter withdraw funds reserved against open policies.
- **Oracle-agnostic.** The same protocol settles policies against Charli3, Orcfax, or the AegisSelf
  publisher feed — chosen per policy, with no validator redeploy to add a provider.
- **A public risk primitive.** Aggregate demand across strikes and durations forms an on-chain
  "fear gauge" any protocol can read via reference inputs.

## How it works

Aegis is a single Aiken project that compiles to **three Plutus V3 scripts**, connected by a
build-sign-submit transaction flow:

```
   Wallet (CIP-30)                Frontend
        |  sign                       |  REST
        v                             v
   Cardano L1   <----- submit -----  API (tx build, indexing)
   policy / pool / lp_token          Charli3 · Orcfax · AegisSelf
   Aiken validators                  oracle reference inputs (CIP-31)
```

- The **policy validator** governs one UTxO per policy (its datum encodes the strike, coverage,
  premium, window, and oracle).
- The **pool validator** is a single shared UTxO holding all underwriter capital and the solvency
  accounting.
- The **LP minting policy** is parameterized by the pool's script hash, so `aLP` can only be
  minted or burned when the pool itself is consumed in the same transaction.

The oracle UTxO is read as a **reference input** (CIP-31) — never consumed — so a single feed can
back many concurrent claims, and the feed's freshness is checked inside the validator.

For the full datums, redeemers, transaction graphs, and edge cases, see
[How Aegis Works On-Chain](architecture.md). For who participates and why, see
[The Seven-Sided Marketplace](marketplace.md).

## Coverage and oracles

A policy names an `oracle_provider` (Charli3, Orcfax, or AegisSelf) and an `oracle_nft`. At claim
time the validator locates the matching feed by NFT, parses the Charli3-compatible `OracleDatum`,
rejects a stale reading (`oracle_expiry > tx.lower_bound`), and pays out only if the price meets
the strike. Aegis performs no off-chain price-oracle work of its own — it only verifies.

## Treasury give-back

Every fee-bearing Aegis transaction routes a fixed share of the protocol fee into the **Cardano
protocol treasury** via the Conway-era `treasury_donation` body field (CDDL key 22). The pool
validator rejects any such transaction that under-pays, so the contribution is enforced
cryptographically rather than by trust, and shows up as a "Treasury Donation" row on cardanoscan.

## Building on Aegis

Integrators (DEXs, lending protocols, wallets) add one-click coverage with the TypeScript SDK,
[`@fluxpointstudios/aegis-sdk`](https://www.npmjs.com/package/@fluxpointstudios/aegis-sdk). It
exposes quote/compose helpers that return the policy parts to splice into your own transaction, a
network-aware oracle-feed registry, and byte-for-byte datum encoders that mirror the on-chain
types. No backend integration is required to read pool state or quote a premium.

## Status

Aegis is **live on Cardano mainnet** — crash coverage, depeg coverage, CDP protection, the
underwriter pool, and the AEGIS / FEAR index are all in active use, with partner integrations
(SaturnSwap, Surf, Indigo CDP protection) onboarding through the SDK.
