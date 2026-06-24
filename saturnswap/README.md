---
description: SaturnSwap — Cardano-native order-book DEX with cross-chain routing
---

# SaturnSwap

SaturnSwap is a **Cardano-native decentralized exchange** built by Flux Point Studios. It pairs
an on-chain **central limit order book (CLOB)** with liquidity-pool depth, giving traders true
limit orders and market orders on Cardano while keeping custody of their assets at all times.
Beyond Cardano, SaturnSwap routes orders across a dozen external chains through an aggregator
layer, and ships an AI trading assistant for natural-language swaps.

Live at [saturnswap.io](https://saturnswap.io).

## Design goals

- **Non-custodial by construction.** Every trade is a Cardano transaction the user signs in their
  own wallet. SaturnSwap never holds user keys or funds.
- **Real order books on Cardano.** Most Cardano DEXs are pure AMMs. SaturnSwap adds a genuine
  limit-order book so traders can post resting orders at a chosen price, not just swap at spot.
- **One signature, broad reach.** A single connected wallet can trade Cardano-native pairs and,
  through the aggregator, reach liquidity on other chains.
- **Transparent settlement.** Orders settle on-chain against audited validators; balances, fills,
  and liquidity are all verifiable from the ledger.

## How it works

SaturnSwap is a hybrid of three layers — on-chain validators, an off-chain matching/serving
backend, and a web client — connected by a build-sign-submit transaction flow.

```
   Wallet (CIP-30)                Frontend (Next.js)
        |  sign                        |  GraphQL
        v                              v
   Cardano L1  <----- submit -----  Backend (.NET 8)
   saturn_swap / saturn_liquidity   order matching, TX build,
   Aiken validators                 chain indexing (Blockfrost)
```

### On-chain: Aiken validators

The protocol's settlement rules live in two Aiken (Plutus) validators:

| Validator | Responsibility |
|-----------|----------------|
| **`saturn_swap`** | Escrows and settles limit/market orders. Enforces that a filled order pays the maker the agreed price and returns the correct assets, and that cancellations return funds only to the order owner. |
| **`saturn_liquidity`** | Manages liquidity positions and the control/pool UTxOs that back order-book depth, governing deposits, withdrawals, and fee accrual. |

Because Cardano uses the eUTxO model, each open order and liquidity position is its own UTxO with
a datum describing its terms. Settlement consumes those UTxOs and produces outputs that the
validator checks against the order's stated price and ownership.

### Off-chain: matching and serving

A .NET 8 backend exposes a **GraphQL API** (plus REST aggregator/router endpoints) that the client
uses for all reads and transaction construction. It indexes the chain through **Blockfrost**,
persists order-book and pool state in PostgreSQL, and builds the unsigned transactions that
implement each trade. It never signs on the user's behalf.

### Transaction flow

Every state-changing action follows the same non-custodial pattern:

1. **Build** — the client calls a GraphQL mutation (e.g. `createOrderTransaction`,
   `createLiquidityTransaction`); the backend returns an *unsigned* transaction.
2. **Sign** — the user's CIP-30 wallet signs it locally, preserving witness order.
3. **Submit** — the client calls the matching `submit…Transaction` mutation to broadcast it.

## Trading

- **Market orders** — execute immediately against available depth. Minimum **5 ADA** per order.
- **Limit orders** — post a resting order at a chosen price; fills settle on-chain when the market
  reaches it. Minimum **25 ADA** per order (a single source of truth shared by the UI and backend,
  surfaced clearly before submission rather than silently rejected).
- **Liquidity provision** — supply assets to back order-book depth and earn a share of trading fees.

Amounts throughout the API are expressed in **display units** (e.g. `5` = 5 ADA), not lovelace.

## Cross-chain routing

SaturnSwap integrates the **UEX aggregator** to extend reach beyond Cardano. A routing layer
compares SaturnSwap's native order-book pricing against aggregator quotes and settles on the better
path — Cardano-native when SaturnSwap wins, cross-chain when the aggregator does. The integration
spans **12 chains**, with quote → order → status tracking and deposit-address validation handled
end-to-end. Native Cardano settlement always takes precedence when it offers the better execution.

## Aurora — AI trading assistant

SaturnSwap ships **Aurora**, a conversational assistant that turns natural-language requests
("swap 100 ADA for SNEK", "what's trending today") into structured, reviewable trade actions.
Aurora is backed by Flux Point Studios' T-Backend (see the
[T-Backend Developer Guide](../t-backend-developer-guide.md)) and uses live market data
for token discovery. It proposes transactions; the user always reviews and signs.

## Token launches and seasons

- **Liftoff** — a launch surface for new Cardano tokens, with optional support for pre-minted
  policies.
- **Saturn Wars** — ranked trading seasons with a leaderboard across volume, liquidity, and a
  skill rating, where top performers share a portion of protocol fees.

## Wallets

SaturnSwap works with CIP-30 Cardano wallets — including **Eternl, Lace, Nami, and Vespr** —
connected through the Lucid library. Connecting a wallet is read-and-sign only; the application
requests signatures for transactions the user explicitly initiates.

## Security model

- **Self-custody** — keys and funds never leave the user's wallet.
- **Validator-enforced settlement** — fills and cancellations are constrained by the on-chain
  validators, not by backend trust. A malicious or buggy backend cannot move funds against the
  validator rules.
- **Reviewable actions** — every transaction (including AI-proposed ones) is shown before signing.
- **Minimums and guardrails** — order minimums and slippage are validated client-side before a
  wallet prompt, with backend validation as the authoritative check.

## Status

SaturnSwap is **live on Cardano mainnet** at [saturnswap.io](https://saturnswap.io), with the v3
interface (swap, tokens, orders, liquidity, leaderboard, portfolio, and the Aurora AI surface)
in active development.

## Building on SaturnSwap?

Aggregators, routers, and apps can execute swaps through SaturnSwap's order book via the public
GraphQL API — no API key, no on-chain contract work. See
[API Integration (Aggregators)](api-integration.md).
