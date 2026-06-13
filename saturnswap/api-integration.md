---
description: >-
  Integrate SaturnSwap swaps from your own app or aggregator using the public
  GraphQL API — build, sign, submit. No API key, no on-chain contract work.
---

# API Integration (Aggregators)

This guide is for developers building **on top of** SaturnSwap — aggregators, routers, bots,
and front-ends that want to execute Cardano swaps through SaturnSwap's order book without
re-implementing the on-chain transaction logic.

You do **not** build SaturnSwap transactions yourself. You call the public GraphQL API, which
returns a ready-to-sign transaction that already carries the Plutus redeemers and SaturnSwap's
protocol co-signature. Your user signs it in their own wallet, you hand it back, and SaturnSwap
submits it to the chain. Trades remain fully non-custodial.

{% hint style="info" %}
**Why route through the API instead of building txs directly?** SaturnSwap's validators charge a
protocol fee. When a transaction is co-signed by SaturnSwap's authority key (which the API does
for you on every order), the standard protocol fee applies. Transactions built and submitted
**without** that co-signature fall back to the contract's higher anti-bot fee. Routing through the
API gets your users the standard fee automatically — and you inherit any future fee or routing
improvements with no code changes.
{% endhint %}

## Endpoint

```
POST https://api.saturnswap.io/v1/graphql/
Content-Type: application/json
```

No authentication or API key is required for the swap endpoints — they are the same public
operations the SaturnSwap web client uses.

## The flow

Every order is three steps: **create → sign → submit**.

```
createOrderTransaction   ──►  unsigned tx (CBOR hex) + transactionId
        user wallet      ──►  partial-sign (CIP-30)
submitOrderTransaction   ──►  on-chain transactionId
```

1. **Create.** Call `createOrderTransaction` with the order details. You get back a
   `transactionId` and an unsigned transaction (`hexTransaction`).
2. **Sign.** Have the user's wallet partial-sign the transaction (CIP-30 `signTx(hex, true)`),
   then merge the returned witness set back into the transaction. Add **only** the user's
   witness — do not modify the transaction body.
3. **Submit.** Call `submitOrderTransaction` with the **same** `transactionId` and the signed
   transaction. SaturnSwap lifts the user's witness into its stored copy of the transaction
   (which holds the script redeemers and the protocol co-signature) and submits it. You get back
   the on-chain `transactionId`.

{% hint style="warning" %}
**Conway tag-258.** Some wallet CSL builds cannot parse Conway canonical-set encoding (CBOR tag
`258`, bytes `0xD9 0x01 0x02`) and will throw `expected Array got Tag` on `signTx`. If you hit
this, strip the tag-258 prefixes from the transaction bytes before signing. SaturnSwap computes
the `transactionId` over the tag-258-stripped body, so stripping it is expected and does not
invalidate the signature.
{% endhint %}

## Step 1 — `createOrderTransaction`

```graphql
mutation Create($input: CreateOrderTransactionInput!) {
  createOrderTransaction(input: $input) {
    successTransactions { transactionId hexTransaction }
    failTransactions { error { message code } }
    error { message code }
  }
}
```

Example — a **market buy** spending 5 ADA on a pool:

```json
{
  "input": {
    "paymentAddress": "addr1...<the user's wallet address>",
    "marketOrderComponents": [
      {
        "poolId": "<pool uuid>",
        "tokenAmountSell": 5,
        "tokenAmountBuy": 1,
        "marketOrderType": "MARKET_BUY_ORDER",
        "slippage": 2,
        "version": 2
      }
    ]
  }
}
```

`hexTransaction` in the response is the unsigned transaction (CBOR hex). You can batch multiple
components — and even multiple order types — in a single `createOrderTransaction` call.

### Input reference

`CreateOrderTransactionInput`:

| Field | Type | Notes |
| --- | --- | --- |
| `paymentAddress` | `String!` | The user's Cardano wallet address (the taker). |
| `marketOrderComponents` | `[MarketOrderComponent]` | Market (immediate) fills. |
| `limitOrderComponents` | `[LimitOrderComponent]` | Resting limit orders. |
| `cancelComponents` | `[CancelComponent]` | Cancel a resting order by `poolUtxoId`. |

`MarketOrderComponent`:

| Field | Type | Notes |
| --- | --- | --- |
| `poolId` | `String!` | Target pool (see [Pool discovery](#pool-discovery)). |
| `tokenAmountSell` | `Float!` | Amount you **give**, in display units (e.g. `5` = 5 ADA on a buy; `1000` = 1000 tokens on a sell). SaturnSwap scales by the token's decimals. |
| `tokenAmountBuy` | `Float!` | Minimum amount you **receive** (slippage floor). |
| `marketOrderType` | `PoolUtxoType!` | `MARKET_BUY_ORDER` (give ADA, get token) or `MARKET_SELL_ORDER` (give token, get ADA). |
| `slippage` | `Float` | Max slippage as a **percent**, `0.1`–`100`. |
| `version` | `Int` | Use `2`. |

`LimitOrderComponent` mirrors the above with `limitOrderType` = `LIMIT_BUY_ORDER` or
`LIMIT_SELL_ORDER`, `tokenAmountSell`/`tokenAmountBuy` as the exact give/receive amounts, and no
`slippage`.

Order-type enum (`PoolUtxoType`): `LIMIT_BUY_ORDER`, `LIMIT_SELL_ORDER`, `MARKET_BUY_ORDER`,
`MARKET_SELL_ORDER`, `CANCEL`.

## Step 2 — sign in the user's wallet

Using a CIP-30 wallet (Lace, Eternl, Vespr, Tokeo, …):

```js
// hexTransaction came from step 1
const witnessSetHex = await wallet.signTx(hexTransaction, true); // partial = true
const signedHex     = assembleTransaction(hexTransaction, witnessSetHex);
// `assembleTransaction` merges the witness set into the tx (e.g. via lucid-cardano or CSL).
```

Only the user's verification-key witness is added; the transaction body is unchanged.

## Step 3 — `submitOrderTransaction`

```graphql
mutation Submit($input: SubmitOrderTransactionInput!) {
  submitOrderTransaction(input: $input) {
    transactionIds
    error { message code link }
  }
}
```

```json
{
  "input": {
    "paymentAddress": "addr1...<same user address>",
    "successTransactions": [
      { "transactionId": "<from step 1>", "hexTransaction": "<signedHex from step 2>" }
    ]
  }
}
```

`transactionIds` are the on-chain transaction hashes. The `transactionId` you pass **must** match
the one returned by `createOrderTransaction` — it keys SaturnSwap's stored copy of the fully
built transaction.

## Pool discovery

Resolve a token to its `poolId`, and read the live book:

```graphql
query Pools {
  pools(where: { ticker: { eq: "SNEK" } }) {
    nodes { id ticker lp_fee_percent }
  }
}
```

- `poolByTokens(...)` — look up a pool by its token's policy id and asset name.
- `orderBookBuyPoolUtxos` / `orderBookSellPoolUtxos` — the resting bids and asks for a pool.

## Errors

- `createOrderTransaction` returns per-component failures in `failTransactions { error { message code } }`
  and request-level problems in `error { message code }`. Common `code`s include `Slippage`
  (the book moved past your slippage tolerance — widen `slippage` or reduce size),
  `InsufficientBalance`, and `WalletFragmented` (the wallet's ADA is spread across too many UTxOs;
  consolidate with a self-payment).
- `submitOrderTransaction` returns submission errors in `error { message code link }`.

## Reference transaction

A 5-ADA market buy executed end-to-end through this API on Cardano mainnet:
[`ca8de851aa044d09ebb135e28acb70a3dd4753059c5636e0f4470652df76bb23`](https://cardanoscan.io/transaction/ca8de851aa044d09ebb135e28acb70a3dd4753059c5636e0f4470652df76bb23).
