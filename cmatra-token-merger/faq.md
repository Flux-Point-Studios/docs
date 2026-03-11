---
description: >-
  Frequently asked questions about the cMATRA token merger, redemption process,
  rates, legacy rewards, NFTs, and the Materios transition.
---

# FAQ

***

## General

### 1. What is changing?

Seven legacy Cardano assets are being consolidated into cMATRA on Cardano, which is intended to converge into MATRA as the native capital token of the Materios network. Materios also uses MOTRA as a separate non-transferable capacity token for fees and sponsorship.

### 2. Is this still a snapshot merger?

Not in the old sense for normal fungible holders.

The merger now uses a **live surrender-and-redeem model** for public holders. You redeem by actually holding and surrendering eligible legacy assets during the six-month window.

A reference snapshot still exists, but it is used for audit, disclosure, Team-waiver publication, and launch reconciliation.

### 3. Why move away from a pure snapshot-entitlement model?

Because a pure snapshot model makes the old fungible tokens feel like zombie bags once the snapshot is over. A live redeem model is cleaner: the holder who controls the legacy asset during the window can trade it in for cMATRA.

### 4. Why does the reference snapshot still matter?

It still matters for:

* Public auditability
* Team treasury waiver disclosure
* Legacy reward reconciliation
* NFT inventory and CIP-68 filtering
* Launch-package sanity checks

It just is **not** the ordinary fungible entitlement rule anymore.

***

## cMATRA and Materios

### 5. What exactly is cMATRA?

cMATRA is the Cardano-side transitional redemption asset. MATRA is the long-term native capital token on Materios. The goal is one economic supply story, not two unrelated supplies.

### 6. What is MOTRA?

MOTRA is the network capacity token. It is non-transferable, generated from MATRA holdings, decays if unused, and is burned when used for fees. It exists so users do not have to sell MATRA just to use the network.

### 7. Is the supply fixed?

Yes. The current design uses a fixed maximum supply of 1,000,000,000 MATRA/cMATRA with 6 decimals.

### 8. How is the supply split at genesis?

15% is carved out at genesis as a non-circulating validator reserve and 85% is assigned to the public redemption pool. The validator reserve is not a later inflation switch; it is part of the genesis supply plan.

### 9. Why does Materios need a validator reserve?

Because network security has to be funded honestly. MOTRA fees are burned and are not designed to pay validators. Validator incentives therefore come from the MATRA side of the system, using a genesis reserve rather than surprise inflation later.

***

## Dilution and Team Policy

### 10. Am I being diluted?

Not in the sense of post-launch inflation. But yes, compared with an older 100% merger assumption, the public redemption pool is smaller because 15% is now reserved for security.

Materios is addressing that directly instead of pretending the cost does not exist.

### 11. How is the hit being softened for holders?

Three ways:

1. Disclosed Team treasury balances are waived from public redemption.
2. Early redeemers may receive time-limited MOTRA sponsorship or delegated capacity during onboarding.
3. The validator reserve remains locked and non-circulating until released by the protocol reward schedule.

### 12. Is the Team taking the same hit?

The Team is taking the first visible hit through a treasury-waiver policy. Disclosed Team treasury balances are not meant to compete with public redeemers for cMATRA from the public redemption pool.

The current waived balances are:

* **AGENT:** 29,644,656 (FPS DAO: 9,902 + $TALOS: 29,634,754)
* **SHARDS:** 446,969.70 display units (all at FPS DAO)

Since the creation of both on-chain treasuries, there has been a complete lack of independent DAO proposals — the only proposals submitted were those proposed by the FPS Team itself. These funds are being returned to the team where they can be put to better use with more flexibility. All waived balances are published transparently so any holder can verify them on-chain.

***

## Eligibility

### 13. Which legacy assets are included?

AGENT, SHARDS, Flux Point Team Pass, SE Brawlers, Brawl Pass: Enter the Dragon, T1 ADAM Launch Pass, and T2 ADAM Launch Pass.

### 14. Do I need to still hold the old assets to get cMATRA?

Yes.

Under the revised model, normal public redemption requires you to **actually hold and surrender the eligible legacy assets during the window**.

### 15. If I buy AGENT, SHARDS, or an eligible NFT now, can I still redeem later?

Yes, as long as:

* The asset is one of the supported merger assets.
* The redemption window is still open.
* You control the asset at the time you surrender it.

The public path is based on possession and surrender during the window, not just on historical snapshot ownership.

### 16. Do I need to use the same wallet that held the asset at the reference snapshot?

No, not for the ordinary public redemption path.

The important thing is that you control the eligible asset when you surrender it during the window and follow the published redemption instructions.

***

## Rates

### 17. Is AGENT 1:1 with cMATRA?

No.

AGENT is **not** 1:1 with cMATRA, and neither are the other legacy assets. The merger uses a weighted redemption model, not a same-supply shortcut.

### 18. So what is the AGENT or SHARDS conversion rate?

The final fixed fungible redemption rates will be published **before** the redemption window opens, after:

* Legacy staking and dashboard-tracked rewards have been credited or minted into actual redeemable old units.
* Final Team treasury waivers have been published.
* The launch reconciliation package has been completed.

The rate table is fixed before launch, but it is not honest to call it final until supply reconciliation is finished.

### 19. Will the rate move during the six-month window?

No, that is not the intended design.

The goal is a **fixed published rate table for the full window**, not a floating live-price conversion that changes every day.

### 20. How are rates set in principle?

Each asset gets a weighted share of the 850,000,000 public redemption pool based on the published valuation methodology.

Then:

* Fungible rate = asset bucket / final redeemable supply for that asset
* NFT rate = asset bucket / final redeemable NFT count for that collection

### 21. Why are final fungible rates not frozen today?

Because Materios is choosing to materialize valid legacy rewards into actual redeemable old units before launch. Until that is finished, the final redeemable fungible supply is not fully reconciled.

***

## Legacy Rewards

### 22. What happens to legacy staking rewards that were tracked operationally or in dashboards?

Those rewards are intended to be **credited or minted into actual redeemable legacy units before the redemption window opens**.

Once that is done, they redeem through the same public path as any other eligible unit of that asset.

### 23. Will there be a separate manual side-claim for those legacy rewards?

That is not the intended policy. The point of the materialization step is to avoid a split system where some users redeem on-chain and others rely on a special-case spreadsheet queue.

***

## DeFi, LP, and Custody

### 24. What if my AGENT or SHARDS are in an LP farm on Minswap, WingRiders, or another DeFi platform?

You must first **unfarm / withdraw / remove liquidity** so the underlying AGENT and/or SHARDS return to a wallet you control.

The redeemable assets are the underlying supported legacy tokens once they are back in your wallet and surrendered during the redemption window.

Materios may try to coordinate with major venues to make this easier, but holders should not wait until the last minute.

### 25. Do LP tokens redeem?

No.

LP tokens are receipts for liquidity positions, not merger assets. To participate in the merger, you redeem the **underlying AGENT and/or SHARDS**, not the LP token itself.

### 26. What if I remove liquidity and get back less AGENT or SHARDS than I originally put in?

You redeem what you **actually withdraw and control** during the redemption window.

If impermanent loss, fees, or trading activity changed the amount of underlying tokens in the position, the merger does not recreate the old deposit for you.

### 27. What if my assets are on a centralized exchange or in someone else's custody?

You can only safely redeem assets you **actually control** in a supported wallet.

Exchange balances, custodial balances, or third-party managed balances are not directly redeemable through the ordinary public path unless you first move them into self-custody before the deadline.

***

## Redemption Process

### 28. Can I redeem in multiple transactions, or do I need to do it all at once?

The intended design is that you can redeem in **multiple transactions** during the window. The same fixed rate table applies throughout the window once published.

### 29. How do I redeem during the window?

The intended flow is:

1. Connect a supported Cardano wallet.
2. Select eligible legacy assets to redeem.
3. Sign a transaction surrendering those assets to the published merger destination.
4. Receive cMATRA at the fixed published rate.

The launch kit will publish the exact operational instructions.

### 30. What happens to the old assets after I surrender them?

They are not intended to remain in free circulation.

Where possible they may be burned. Where direct burn is not available or practical, they should be permanently locked, quarantined, or otherwise removed from public circulation under the published merger controls.

***

## NFTs

### 31. How are NFTs handled?

Eligible NFTs redeem on an asset-by-asset basis. Only true one-of-one user-side NFT units count. For CIP-68 collections, the user token counts and the reference token does not.

### 32. What if my NFT is listed on a marketplace or held in a script?

You need to regain normal wallet control of the NFT before redeeming it. The right follows control of the asset at redemption time, not just historical ownership.

### 33. Do CIP-68 reference tokens redeem?

No. Only the user token counts for redemption. The reference token does not.

### 34. What if I wait until the last day and the marketplace, DeFi venue, or network is congested?

Do not wait until the last day.

The six-month window is meant to give holders enough time to recover assets from farms, marketplaces, or other venues. Materios cannot guarantee third-party platform responsiveness or network conditions right at the deadline.

***

## After the Window

### 35. What happens after six months?

The public redemption window closes. Unredeemed legacy assets no longer have an open public trade-in path unless an explicit extension or exceptional remedy is formally announced.

### 36. What happens to any unreleased cMATRA from the 850M public redemption pool after the deadline?

That will follow the published post-deadline policy in the final launch package. It should not be left vague or handled by surprise later.

***

## Materios Onboarding

### 37. Do I need to bridge to Materios immediately after redeeming?

No. The intended flow is Cardano-first. Users redeem into cMATRA on Cardano first, then complete address association and later bridge or lock into Materios as the onboarding flow matures.

### 38. Will cMATRA and MATRA be separate supplies?

No. The intended model is one economic supply story. cMATRA is the transitional Cardano-side representation and MATRA is the Materios-native capital token.

***

## Validators

### 39. How are validators expected to be rewarded?

From the MATRA validator reserve, not from burned MOTRA fees. The reserve is meant to release over time according to the protocol's validator reward schedule.

### 40. Can unused validator reserve be repurposed later?

Not casually. The reserve is not discretionary operating capital. Any later reassignment of clearly surplus reserve should require governance and should be communicated explicitly.

***

## Market and Valuation

### 41. Do we know the starting market cap of MATRA?

Materios does **not** set a guaranteed opening market cap. Market cap depends on what the market decides to pay once trading starts and on how much supply is actually circulating.

What Materios can publish in advance is the fixed supply model and the reference valuation package used to derive redemption weights.

***

## Authority

### 42. Which policy document is authoritative if old docs or messages conflict?

The authoritative launch package is intended to be:

* The final litepaper
* The final eligibility rules
* The final FAQ
* The final fixed rate table
* The legacy reward reconciliation / materialization package
* The final Team waiver publication
* The final redemption instructions

Earlier snapshot-era drafts, stale reserve analyses, or Discord shorthand should not override the final launch package.

### 43. Is this document final?

It is a public / governance draft aligned to the current policy direction. The core model is now much clearer than before, but the final launch package still needs to lock:

* Legacy reward materialization
* Final Team waiver amounts
* Final fixed rate table
* Final post-deadline handling

***

**Version:** 6.1 | **Date:** March 11, 2026 | **Status:** Public / governance draft\
**Companion documents:** Litepaper, eligibility rules, fixed rate table, legacy reward reconciliation package, validator incentives spec
