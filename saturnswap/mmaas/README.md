---
description: >-
  Market Making as a Service — a continuously quoted two-sided book on your
  token, resting at an address only your own key can empty.
---

# What MMaaS is

A new Cardano token usually has no resting bid and no resting ask. Someone who wants to buy has
nothing to lift; someone who wants to sell has nothing to hit. The usual fix is to hand a market
maker your inventory and hope.

**MMaaS is that service without the handover.** You keep the keys. SaturnSwap's keeper posts a
two-way order for you on SaturnSwap's on-chain order book — a canonical cardano-swaps two-way
order carrying both legs at once, a bid and an ask — and reprices it around a live mid as the
market moves, so both sides stay quoted instead of going stale.

Setup is self-serve, in your browser, at [saturnswap.io/v3/mmaas](https://saturnswap.io/v3/mmaas).
Every transaction is built and signed by your own wallet; there is no form to fill in and no
approval to wait for. One step is still manual, and it is named in
[What you sign](#what-you-sign).

## Check which validator your book uses

The guarantees below describe the current `18d2246d…` generation. Existing `adc2a7f1…`
books retain their original code: they lack the input-pair preservation check and
staking-yield fee exclusion. Read [Validator generations](validator-generations.md)
to verify your funded address and understand the client-signed migration path.

## Custody: what we can and cannot do

Your inventory rests at a two-way order address whose **stake credential is a Plutus validator,
`maker_stake_bound`, compiled from your own parameters**. It governs owner actions;
permissionless taker fills also follow the DEX spend validator. These are separate paths.

Its `withdraw` handler is a two-branch `or`. The first branch is just:

```
list.has(self.extra_signatories, client_owner_vkh)
```

Your signature, alone, unconditionally — it reads nothing else. Spending a resting order requires
that credential to appear in the transaction's withdrawals, so **you can sign cancellations
and take the recovered funds anywhere you like**, in batches that fit transaction limits. No cooperation from
us, no notice, and we never hold that key. Mint, spend, vote and propose under the same hash are
client-only too, and every certificate except a bare registration needs your signature — so we
cannot deregister your credential to pocket its deposit, and we cannot delegate your stake.

The second branch is what SaturnSwap can do. It requires our bot key **and** a per-asset
conservation check: everything spent from your order addresses must land either back at an order
address carrying the same stake credential and input pair with an in-band, decodable datum, or in an
exact-address, datum-free output to the payout address your ceremony names — plus a bounded,
ADA-only fee leg at the published fee address, bounded in total so splitting it across outputs
does not multiply it. Anything else fails the transaction.

So, stated plainly: **we can reprice your order and cancel it back to your payout address.**
Bot transactions must preserve your assets within the permitted outputs, including the bounded
ADA fee leg described above. These checks are enforced by consensus.

The same check is why the gas is ours. Your order's own value can only reach those three
destinations, so a transaction that paid its network fee out of your inventory would come up
short and be rejected. Every reprice we sign, we fund.

## The band

You set two prices, and the validator enforces them on every quote:

- **`min_asset1_price` — the most we may bid.** The keeper can never pay more than this in ADA
  per token.
- **`min_asset2_price` — the least we may ask.** The keeper can never sell your token below this.

Both are checked by `rational_geq` against the continuation datum, so an out-of-band quote is not
a policy we promise to follow — it is a transaction the chain refuses. The validator also refuses
a **crossed** band: the ask floor must sit at or above the bid ceiling, which is what stops a
filler round-tripping your own book for a profit.

The band is fixed at setup, because it is one of the nine parameters the address is derived from.
Changing it means a new instance at a new address — see
[Risks, limits, and how to leave](risks-and-exit.md#one-band-per-instance).

## What it costs

|  |  |
|---|---|
| Credential deposit | **2 ADA**, the Cardano stake-credential deposit. Refunded in full when you retire the credential. |
| Service fee | Default **20 basis points (0.20%) of eligible settled ADA notional**, subject to the client's billing grant and any configured cap. Metered by the backend and collected from a **separate prepaid ADA fee channel**. This is not a deduction from each swap or the inventory validator's payout fee. |
| Prepaid balance | You fund the fee channel separately from trading inventory. The operator can collect from it under the channel's signing policy; you can reclaim the remaining balance with your own key. |
| Network fees | You pay transactions you sign. The operator funds keeper reprices and inventory cancellations. A fee-channel collection pays its network fee from the channel balance, in addition to the amount collected. |

### How service billing works

The backend values an eligible fill by the ADA that moved, applies the grant's fee rate, and
limits the accrued amount by its configured cap. It checks consent lineage, grant timing and
finality, excludes known operator fills using the transaction's full signer set, and holds
historical fills whose signer evidence is missing. An unfamiliar signer is **unattributed**;
it does not prove the counterparty is independent. Token-to-token fills with no ADA leg have
no ADA notional under this meter.

The fee collector draws the uncollected amount from your prepaid channel. The two-operator-key
channel has the native-script policy **either both operator keys together, or your client key
alone**. A legacy channel instead permits **either one operator key, or your client key alone**.
Verify which script and keys your channel uses before funding it.

**The prepaid balance has different protections from trading inventory.** The native script
checks signatures. It does not check an invoice, a fee rate, a period cap or how much an
authorised operator branch spends. Billing eligibility and collection limits are software and
co-signer policy controls. The inventory validator's `max_fee_bps = 500` does **not** cap a
fee-channel draw. Keep the channel balance consistent with the exposure you accept.

Signing the nine-parameter possession challenge authorises keeper consent for that ceremony;
it does not itself enrol a billing grant or place the grant's commercial terms on chain.

### The separate inventory-validator fee capability

The inventory validator permits a bounded, ADA-only fee output at its published fee address.
Its bound is calculated on realised ADA payout plus that fee, with `max_fee_bps = 500` in the
published parameters. A beacon-burning close requires this **inventory fee** to be zero.
The `18d2246d…` generation subtracts withdrawn staking rewards from that fee basis; the older
`adc2a7f1…` generation does not. These rules describe the inventory transaction, not commercial
fee-channel collection; closing a book does not erase already accrued service fees.

The operator-side ceremony parameters are published under **Check us, don't trust us** on
[saturnswap.io/v3/mmaas](https://saturnswap.io/v3/mmaas). Check the
[validator generation](validator-generations.md) of the address you actually funded.

## What your token needs

**A live market, on two feeds.** Prices come from two independent public sources — bending.ai and
GeckoTerminal — read by policy id and asset name, with a divergence breaker between them.

- Setting up a band needs **both** feeds to answer and to agree within **5%**. One feed is not
  enough: a band is permanent, and a mid read from a single source is a permanent decision made
  on an unchecked number.
- Once you are running, the keeper applies the same rule each round. A pair sourced from feeds is
  priced by its feeds **or not at all** — a feed outage or a divergence trip means your book is
  not quoted that round rather than quoted badly.
- An operator-set constant mid is **refused outright on mainnet** by the keeper's own code. It
  would stub both feeds with one number and defeat the breaker.

A token listed on neither feed cannot currently be quoted. You can still build a band by hand,
but nothing will work it.

**Published decimals.** Your token's decimals are a scale exponent baked permanently into your
order address, so the page resolves them from the token's own CIP-68 on-chain metadata or the
CIP-26 token registry, and refuses to continue if neither states them or the two disagree. If
your token publishes decimals nowhere, register it before you come back.

## What you sign

Three wallet prompts, in this order — and one more if you are moving an existing book:

1. **Prove** the wallet is yours: a CIP-30 signature, no transaction and no fee.
2. **Register** your credential on chain (the 2 ADA deposit).
3. **Fund** the order — your inventory into your own order address.

**The order matters.** That first signature is your consent to be market-made, and it travels
*inside* the registration transaction as metadata under label 8747. Register first and it carries
none — and a certificate cannot be amended afterwards, so the only way to add it later is a new
instance at a new address.

You publish it yourself, in your own transaction. There is nothing to send us. The keeper reads it
back off the chain and checks it against your ceremony: the signature must come from the payout
address your parameters baked in, and the challenge it carries must be the one those nine
parameters hash to. Without a proof that passes both, the keeper declines to quote your book — your
order still rests on the public book, fillable by anyone at the prices you set, and nobody
reprices it.

Next: [Setting up a book](setting-up-a-book.md) ·
[Risks, limits, and how to leave](risks-and-exit.md)
