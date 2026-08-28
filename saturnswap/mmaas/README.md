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

## Custody: what we can and cannot do

Your inventory rests at a two-way order address whose **stake credential is a Plutus validator,
`maker_stake_bound`, compiled from your own parameters**. That validator is the whole custody
story, and it is worth reading precisely.

Its `withdraw` handler is a two-branch `or`. The first branch is just:

```
list.has(self.extra_signatories, client_owner_vkh)
```

Your signature, alone, unconditionally — it reads nothing else. Spending a resting order requires
that credential to appear in the transaction's withdrawals, so **one transaction you sign can
cancel every order at that address and take the funds anywhere you like.** No cooperation from
us, no notice, and we never hold that key. Mint, spend, vote and propose under the same hash are
client-only too, and every certificate except a bare registration needs your signature — so we
cannot deregister your credential to pocket its deposit, and we cannot delegate your stake.

The second branch is what SaturnSwap can do. It requires our bot key **and** a per-asset
conservation check: everything spent from your order addresses must land either back at an order
address carrying the same stake credential with an in-band, decodable datum, or in an
exact-address, datum-free output to the payout address your ceremony names — plus a bounded,
ADA-only fee leg at the published fee address, bounded in total so splitting it across outputs
does not multiply it. Anything else fails the transaction.

So, stated plainly: **we can reprice your order and we can cancel it back to you. We cannot send
your value to ourselves, to a third party, or to any address that is not yours.** That is
enforced by consensus, not by our conduct.

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
| Our fee | **20 basis points (0.20%) of what a transaction realizes** — the ADA it pays out to you, plus the fee itself. Taken as a separate, **ADA-only** output at the published fee address, so a fee can never be taken in your token. Bounded on chain: the validator declares `max_fee_bps = 500` and refuses to act above it. |
| Fee on a close | **None.** A transaction that burns a beacon — how a cardano-swaps order closes — must take a fee of exactly zero. You are charged for a position being run, not for getting one back. |
| Fee on staking yield | **None.** Any rewards withdrawn in the same transaction are subtracted from the fee basis before the rate is applied. |
| Network fees | You pay the fee on the transactions you sign. We pay it on every reprice we sign. |

The five operator-side parameters — fee address, fee rate, bot key hash, DEX validator hash and
beacon policy id — are published under **Check us, don't trust us** on
[saturnswap.io/v3/mmaas](https://saturnswap.io/v3/mmaas), each with the chain query that confirms
it, so you can check them against the ledger rather than take them from us.

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
