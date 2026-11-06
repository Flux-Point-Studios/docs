---
description: Who takes part in the Sovereign Book, what each role pays and earns, and what each role can and cannot do.
---

# Who Participates

The Sovereign Book has two sides that exchange risk, and a set of services that keep it running. No
service can move an underwriter's capital anywhere except home, and none can redirect a payout.

## Buyers

A buyer pays a premium for a fixed payout if a price reaches a strike during the term.

- **Pays:** the premium, the fee, the transaction fee and a deposit of about 2.1 tADA in the P-leg.
  Whoever submits the settlement can take that deposit as change.
- **Receives:** the full coverage on a Claim, sent to the buyer's own address.
- **Can:** cancel within 30 minutes of the start for 90% of the premium, while the reading is out of
  the money.
- **Risks:** the term ends with no trigger and the premium is spent. A Claim also needs a fresh
  reading that shows the trigger, and the claim wait of 1 hour must have passed.

The contract does not check whether the buyer holds a loan or the asset. A borrower can use a Barrier
position as liquidation protection by choosing a strike above the liquidation price, and anyone else
can hold the same position as a synthetic put.

## Underwriters

An underwriter posts offers at the book script under their own stake credential.

- **Puts up:** ADA, up to the offer's capital. A position locks only what it fills.
- **Earns:** the premium on each position that expires, 10% of the premium on a cancelled position,
  and staking rewards. The capital stays under the underwriter's stake credential both as an offer
  and as a leg, so it keeps its delegation.
- **Controls:** feeds, classes, depth and term ranges, fill sizes, price sheet, load and expiry. The
  owner can cancel an unfilled offer at any time.
- **Risks:** a leg pays out in full if its position triggers. A leg is locked until its position
  settles, which for an unclaimed position is 48 hours after expiry.

Capital is not pooled. Each leg backs one position, and a loss on one underwriter's leg cannot reach
another underwriter.

## Agents and the hosted maker

An offer can name an agent. The agent can withdraw the offer to its owner or narrow its terms, and in
approval mode it must witness every fill. It cannot take the capital or widen the terms.

The hosted maker is one such agent. It reads the offers that name its agent key, and under a halt it
sends them home, its clients' offers first. It pays its own transaction fees from a separate float.

## The relay

The relay allocates each sale across admitted offers and presents the body for the buyer to sign. It
ranks by price, then by when an offer was posted. Its mark on each sale records which allocation
produced it. Validators do not depend on the relay.

## The publisher and the observer

The publisher writes the price feeds. A signing core refuses a bundle unless at least two exchange
sources agree with it and its trailing low can be found in the observer's Pyth-signed, on-chain
anchored history.

## The keeper

The hosted keeper submits Claims and Expires. Settlement is permissionless, so a buyer or an
underwriter can submit the same transaction. The validators hold every settlement to the full amount,
so the keeper keeps none of the coverage.

## The quorum

A 2-of-3 quorum can return offers to their owners in an emergency, short by at most 0.6 tADA to pay
the batch's fee. A transaction it signs may contain nothing except those returns.

## Integrators

DEXs and lending protocols sell cover at the point of use. SaturnSwap swap-plus-insure and FluidTokens
V4 liquidation cover are in progress against the Book. See [Integrations](README.md#integrations) for
the state of each.
