---
description: How Aegis cover protects a borrower's position before it reaches liquidation or recovery, what can be covered today, and what an integrating protocol provides.
---

# Cover for Lending Positions

A borrower who has pledged collateral can buy Aegis cover that pays out before the position reaches
liquidation or recovery. The cover is a Barrier on the collateral's price. Its strike sits above the
price at which the position gets closed, so the payout arrives while the borrower still has time to
top up or repay.

The cover is parametric. It pays a fixed amount when the price reading reaches the strike during the
term. It does not wait for a liquidation and does not ask anyone to judge a loss.

## How a position maps to a cover

| Cover term | Set from the position |
|------------|-----------------------|
| Feed | The collateral's price feed, ideally the same price the lending protocol closes positions on |
| Strike | A price above the position's liquidation or recovery price, with a margin the borrower chooses |
| Coverage | The amount the borrower wants paid, for example what it takes to bring the position back to a safe ratio |
| Term | At least 7 days and at most 180. The borrower renews by buying again |
| Payout | To the borrower's own wallet address, in ADA, or in USDCx on the next generation's USDCx instance |

Two rules bind the strike on chain:

- It must sit **at least 15% under** the lower of the current reading and its trailing low. A
  position that is already within 15% of its strike price cannot be covered at that strike. Cover is
  for positions that are healthy today.
- It must sit **under the trailing low**, so a buyer cannot purchase cover on a fall that has already
  happened.

Claims are permissionless. The hosted keeper submits them, and the payout can only go to the buyer.
A buyer can cancel in the first 30 minutes for a 90% refund, as long as the cover is not already in
the money.

## Which collateral can be covered

| Collateral | State |
|------------|-------|
| ADA | Barrier cover on ADA/USD is live on preprod, and on both preview instances. A Claim has paid on a real ADA fall, see [Preprod Proof](preprod-proof.md#settlements) |
| BTC, ETH | On the preprod feed table. No underwriter has posted offers for them yet |
| USDC, USDT | Depeg cover on USDC is live on preprod. USDT is on the next generation's table, and one sale on it on preview was cancelled by its buyer |
| sUSDrf | Needs a new feed. Its price is RealFi's staking-vault exchange rate times USDrf's USD price. The rate can be computed from the vault's own UTxO on chain, and the USDrf leg from the SundaeSwap USDCx/USDrf pool. Neither leg is published yet, and the pool leg needs a time condition so one large trade cannot move it |
| LP tokens and other receipts | Case by case. Each needs a price source both sides accept |

A new feed on the next generation is a governed row in the asset table: an authenticated reading,
a watcher and a signer path that can attest it. A row was added this way on preview, see
[A governed table change](preview-proof.md#a-governed-table-change).

The strongest trigger is the price the lending protocol itself acts on. If a protocol closes
positions on its own mark, Aegis can read that mark through its publisher, or the protocol can sign
its marks and the cover verifies the signature.

## How a borrower buys it

1. The lending app links to Aegis with the position's terms filled in: feed, strike, coverage and
   term.
2. Aegis quotes the cover from the underwriters' offers. The quote is a read and reserves nothing.
3. The borrower signs one transaction in a CIP-30 wallet. The borrower's key is the insured party.

An app that builds its own transactions can place the cover sale beside its own steps in one
transaction, as SaturnSwap does with a swap. On preview, one transaction has opened a FluidTokens
loan and sold its cover to the borrower. The quote and build routes are part of the hosted API.
A Book client for the TypeScript SDK is not published yet.

## Cover on the liquidation event

The next generation can also cover the liquidation itself. A loan is bound to an event slot on the
asset table, and the publisher signs the slot's reading: 1.0 while the loan is open and 0.1 once the
lending protocol's own liquidation of that loan lands. A cover on the slot pays its full coverage
when the loan is liquidated during the term.

This pays after the position is closed, where a Barrier pays before, and it reads no price feed for
the collateral. On preview, a FluidTokens V4 loan was liquidated by its lender and the cover on it
paid 30 tADA to the borrower, see
[FluidTokens liquidation cover](preview-proof.md#fluidtokens-liquidation-cover).

## Who underwrites it

Underwriters post standing offers, each with its own feeds, strike depths, terms, sizes and price
sheet. Each integrated protocol is its own risk bucket. An underwriter decides whether to back cover
on that protocol's positions and on what terms. Aegis does not pool underwriters or choose for them.

A protocol's treasury can underwrite cover on its own venue. On the next generation, a treasury that
holds stablecoins can post offers on the USDCx instance, and an offer can be owned by a script, so a
multisig or a vault can be the owner.

## Where it runs

| Network | State |
|---------|-------|
| Preprod | ADA Barrier cover on the current generation, with test ADA |
| Preview | The next generation, live since 2026-11-18. Barrier cover on ADA/USD on both instances, and event cover on FluidTokens loans |
| Mainnet | After the next generation's audit. No generation of the Book runs on mainnet today |

## What we need from an integrating protocol

- The liquidation or recovery rule, and the price it reads.
- The collateral assets and the source of each price.
- Whether the protocol would publish or sign its marks.
- Typical position sizes and how long borrowers keep them open.
- Where a "protect this position" action fits in the protocol's interface.

Reach us on [Discord](https://discord.gg/MfYUMnfrJM).
