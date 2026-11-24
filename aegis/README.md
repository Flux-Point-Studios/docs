---
description: Aegis is parametric cover on Cardano, sold from the Sovereign Book. Status, what it does and where it runs today.
---

# Aegis

Aegis is **parametric cover on Cardano**. A buyer pays a premium and receives a fixed payout if a
signed price reading reaches the strike during the term. Underwriters post standing offers from
their own stake credentials, and each sale moves the full coverage amount into the position's own
UTxOs. Settlement is decided by Aiken validators and a reading they check themselves. Anyone can
submit a claim, and it pays only the buyer.

The current product is the **Sovereign Book**. Its current generation runs on Cardano preprod and its
next generation on Cardano preview, both with test ADA.

## Status

| Network | State |
|---------|-------|
| Preprod | The Sovereign Book is deployed and has run the full lifecycle: Barrier cover on ADA and Depeg cover on USDC sold, buyer cancels, five Expires, and one Claim paid on a real ADA fall through a sold strike. A SaturnSwap swap sold with its cover in one transaction is on chain too. See [Preprod Proof](preprod-proof.md). |
| Preview | The next generation is live, with its ADA and USDCx instances, since 2026-11-18. Ten sales, three buyer cancels, one Claim paid on a FluidTokens liquidation and one Expire are on chain, as is a row added to the asset table through governance. See [Preview Proof](preview-proof.md). |
| Mainnet | New cover sales have been paused since 2026-09-04 for the move to the Book. The prior vault generation remains deployed, and its liquidity providers can withdraw. |
| Audit | No generation of the Book has been independently audited yet. The next generation will be audited at its code freeze, before any mainnet launch. |

Preprod custody is deliberately relaxed for this stage, and signer hardening follows. Nothing on
preprod carries real value.

## What the Book does

- **Barrier cover** on ADA, BTC and ETH pays if the price reaches a strike at least 15% under the
  lower of the current reading and its trailing low.
- **Depeg cover** on USDC pays if the feed falls to a strike between 50% and 95% of the $1 peg. It
  can be sold only while the peg reads at 99% or better.
- **Underwriting** is posting an offer: your capital, your price sheet, your limits. The capital
  stays under your stake credential while it waits and while it backs a position, so it keeps
  earning staking rewards and keeps its delegation.

Terms run at least 7 days. The reference price sheets on preprod sell 7 to 45 days. Each position
can draw on up to 20 offers.

## How a sale works

```
 Underwriters                     Buyer (CIP-30 wallet)
   offers at the book script        |  signs
   each under its own stake         v
        |                     +--------------+     reference inputs:
        +-------------------> |  sale tx     | <-- price sheets, the feed
                              +--------------+     through the observer
                                |    |     |
                 P-leg: policy  |    |     |  fee output
                 C-legs: one per offer, each holding exactly its coverage
```

1. The relay allocates the sale across admitted offers, cheapest first, and marks the body it
   presents.
2. The buyer signs. The position script checks every offer the sale spends against that offer's
   own terms and price sheet.
3. Each contributing offer moves into a C-leg that holds exactly its share of the coverage and
   keeps its owner's stake credential. Any remainder of the offer goes back to the book.
4. The position ends one of three ways: a Claim pays the buyer the coverage, an Expire returns each
   leg to its owner, or a buyer cancel in the first 30 minutes refunds 90% of the premium.

The full rules are in [How Aegis Works On-Chain](architecture.md). The roles are in
[Who Participates](marketplace.md).

## Design properties

- **Fully funded at sale.** A position holds its whole coverage from the moment it is sold. There
  is no shared pool to run short, and one underwriter's position cannot draw on another's capital.
- **Parametric.** A payout depends on the strike and term pinned at sale and on a fresh signed
  reading. No person adjudicates a claim.
- **Underwriters set their own terms.** An offer names its feeds, depths, terms, fill sizes, price
  sheet and expiry. An agent the owner names can only send the offer home or narrow its terms.
- **Permissionless settlement.** Anyone can submit a Claim or an Expire. The hosted keeper does so
  as a service, and it cannot change who gets paid.

## Fees

This generation sends at least 1% of the premium to one fixed address set at deployment, enforced
by the position script at sale. A buyer cancel does not return it. On preprod, the output also has
to meet the ledger's minimum UTxO, which is why the sales on the proof page show about 1.14 tADA.

This generation makes no Cardano Treasury donation, and its feed table is a compiled script
parameter, so adding an asset means a new generation. Both change in the next generation.

## The next generation

The next generation of the Book is live on Cardano preview since 2026-11-18. An ADA-only instance
on preprod follows. It keeps the model above: fully funded positions, underwriters' own offers and
permissionless settlement. What changes:

- **A governed asset table.** Feeds and event slots are rows in an on-chain table, changed through
  a timelocked governance path, so a new asset no longer needs a new generation.
- **Two instances.** One sells cover paid in ADA, the other cover paid in USDCx.
- **Fees.** 2% of the premium to the protocol, plus a 0.5% share for the Cardano Treasury, both
  enforced by the position script.
- **Script-owned offers.** An offer can be owned by a script as well as a key, so a vault or
  a DAO treasury can underwrite.
- **Minimum premium.** 20 ADA, or 10 USDCx on the USDCx instance.
- **Receipts as backing.** On the USDCx instance an offer can hold Liqwid qUSDCx receipts, valued at
  the market's exchange rate with a 15% haircut, so capital supplied to Liqwid can back cover
  without being withdrawn.
- **Event cover.** An event slot on the table carries a signed reading of an outside event, such as
  the liquidation of a named FluidTokens loan, and cover on it pays when the event happens.

It will be audited at its code freeze. Mainnet comes after that audit. Its transactions are on
[Preview Proof](preview-proof.md).

## Integrations

| Integration | State |
|-------------|-------|
| SaturnSwap swap-plus-insure | Live on preprod. One transaction fills a SaturnSwap V3 order and sells USDC Depeg cover from the Book, signed in a mobile wallet. See [The Depeg sales](preprod-proof.md#the-depeg-sales). The SaturnSwap interface for it is not yet on SaturnSwap's public site |
| FluidTokens V4 liquidation cover | Live on preview. A loan's liquidation through FluidTokens paid a 30 tADA Claim, and a loan and its cover have opened in one transaction. See [FluidTokens liquidation cover](preview-proof.md#fluidtokens-liquidation-cover) |
| Indigo | Sold on the prior generation, not yet relaunched on the Book |
| Liqwid | Live on preview. USDCx offers hold qUSDCx receipts from Liqwid's preview market as backing, and five sales have drawn on them. See [Liqwid receipts as backing](preview-proof.md#liqwid-receipts-as-backing). No Claim has paid in receipts yet |
| Babel fees | Live on preview. A buyer pays the premium and every ADA part of a sale in USDCx through a tank built on FluidTokens' Aquarium validators. See [Paying in USDCx with no ADA](preview-proof.md#paying-in-usdcx-with-no-ada) |
| Surf | Surf's services relaunched in November 2026. Re-integration against the Book to follow, with no date set |
| Lending and credit lines | Cover that pays before a borrower's position reaches liquidation or recovery. Design stage, see [Cover for Lending Positions](lending-cover.md) |

The published TypeScript SDK,
[`@fluxpointstudios/aegis-sdk`](https://www.npmjs.com/package/@fluxpointstudios/aegis-sdk), targets
the prior generation. Its re-cut against the Book is part of the SaturnSwap work.

## Earlier generations

Aegis began as a single shared pool with `aLP` tokens (V4 to V7), then moved to a stable vault with
M-of-N registry governance and a 72-hour timelock (V8). Both funded cover from pooled capital. The
Book replaces pooled underwriting: capital stays with the underwriter that posted it, and each
position is funded separately. The prior vault is summarised in
[How Aegis Works On-Chain](architecture.md#earlier-generations).

Aegis has no token.
