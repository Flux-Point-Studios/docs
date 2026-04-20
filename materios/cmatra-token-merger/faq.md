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

27.75% (277.5M) is carved out at genesis as a non-circulating **Network Incentives Reserve** split into five sub-buckets, and 72.25% (722.5M) is assigned to the **Public Redemption Pool**. The reserve is not a later inflation switch; it is part of the genesis supply plan.

The five sub-buckets are:

* **Validator Emissions — 115M (11.5%)** — block-production rewards; includes Cardano SPO cross-validation rewards.
* **Attestor Emissions — 65M (6.5%)** — threshold-attestation rewards + bond/slash economics.
* **Ecosystem Treasury — 40M (4%)** — grants, ecosystem dev funding, LP Loyalty Bonus, governance-directed programs.
* **Strategic Allocation — 30M (3%)** — strategic/institutional partners, 12-month cliff + 36-month linear vesting.
* **Liquidity — 27.5M (2.75%)** — 5M bridge peg reserve, 17.5M Protocol-Owned DEX Liquidity, 5M maker rebates on CLOB.

### 9. Why does Materios need a Network Incentives Reserve?

Because network security, launch liquidity, and ecosystem growth all have to be funded honestly. MOTRA fees are burned and are not designed to pay validators. The Network Incentives Reserve handles all five obligations in transparent sub-buckets rather than surprise inflation later:

1. **Validator rewards** come from the 115M Validator Emissions sub-bucket (supplemented by 40% of transaction fees routed to the block-author pot).
2. **Attestor rewards** come from the 65M Attestor Emissions sub-bucket (supplemented by 30% of transaction fees).
3. **Ecosystem growth** — grants, game integrations, LP Loyalty Bonus — comes from the 40M Ecosystem Treasury.
4. **Strategic capital** — the 30M Strategic Allocation funds POL seeding, security audits, and team runway beyond the redemption window, under strict vesting.
5. **Launch liquidity** — the 27.5M Liquidity sub-bucket seeds the CLOB and AMM pairs, funds maker rebates, and holds the bridge peg reserve.

Once the network is running under fee load, ongoing transaction fees recycle back into these reserves (40% author / 30% attestor / 20% treasury / 10% burn), so the reserves are designed as a first-year runway rather than a permanent ceiling.

***

## Dilution and Team Policy

### 10. Am I being diluted?

Yes, by approximately **15% relative to the v3.3 published rates** — a post-reconciliation correction, not a post-launch inflation. The v5.1 rate table is the v3.3 rates multiplied by 0.85 across the board, which moves the public pool from 850M cMATRA to 722.5M and re-targets the freed 127.5M cMATRA into the five Network Incentives Reserve sub-buckets that were missing (or under-allocated) in v3.3.

This falls squarely within the governance-approved correction clause in the README's "Window mechanics" section: *"The rate table remains fixed for the full window unless a documented governance-approved correction is required for a launch bug or reconciliation error."* The v3.3 rates were mathematically inconsistent with the reserve architecture already committed in runtime code (attestor emissions were unbudgeted) and left zero capacity for ecosystem treasury, strategic partners, or launch liquidity. Shipping v3.3 unchanged would have meant either re-breaking the runtime or launching without POL — both worse outcomes than a proportional 15% correction before the window opens.

The Team takes exactly the same 15% cut (see Q12 below). No holder class is disadvantaged relative to any other.

### 10a. What changed vs v3.3?

| Item                              | v3.3 (March 2026)   | v5.1 (April 2026)                    |
| --------------------------------- | ------------------- | ------------------------------------ |
| Public Redemption Pool            | 850M (85%)          | 722.5M (72.25%)                      |
| Reserve                           | 150M validator only | 277.5M across 5 sub-buckets          |
| 1 AGENT →                         | ~0.5446 cMATRA      | ~0.4629 cMATRA                       |
| 1 SHARDS →                        | ~34.0 cMATRA        | ~28.9 cMATRA                         |
| 1 Flux Point Team Pass →          | ~81,495 cMATRA      | ~69,271 cMATRA                       |
| 1 SE Brawler →                    | ~41,523 cMATRA      | ~35,294 cMATRA                       |
| 1 Brawl Pass: ED →                | ~132,117 cMATRA     | ~112,300 cMATRA                      |
| 1 T1 ADAM Pass →                  | ~3,755,146 cMATRA   | ~3,191,874 cMATRA                    |
| 1 T2 ADAM Pass →                  | ~221,576 cMATRA     | ~188,339 cMATRA                      |
| Team carve total                  | ~31.3M cMATRA       | ~26.6M cMATRA                        |
| Attestor Emissions sub-bucket     | not allocated       | 65M (resolves code↔docs inconsistency) |
| Ecosystem Treasury sub-bucket     | not allocated       | 40M                                  |
| Strategic Allocation sub-bucket   | not allocated       | 30M (12mo cliff + 36mo linear)       |
| Liquidity sub-bucket              | not allocated       | 27.5M (POL + bridge + maker rebates) |

The TWAP-derived **weights** between assets are unchanged — only the absolute cMATRA output is reduced proportionally.

### 10b. How is the post-reconciliation dilution handled in practice?

* **Rates recompute by multiplier only.** Each asset's cMATRA-per-unit rate becomes (v3.3 rate) × 0.85. No asset class is singled out.
* **Team carve recomputes too.** 31.3M cMATRA → 26.6M cMATRA automatically — the Team takes the same haircut.
* **Weights unchanged.** The relative share that AGENT, SHARDS, and each NFT collection receive of the pool is identical to v3.3 (TWAP weights locked before v5.1).
* **Bucket totals verified.** The sum of all seven asset buckets is exactly 722.5M cMATRA in base units (audited in `rate_table_cmatra.json`).
* **No retroactive change after launch.** Once v5.1 is published as the final fixed table, it is locked for the full 6-month redemption window.

### 11. How is the hit being softened for holders?

Four ways:

1. **Disclosed Team treasury balances are waived** from public redemption (29.6M AGENT + 447K SHARDS). The rate denominator already excludes these, so the carve-out does not further dilute public holders.
2. **LP Loyalty Bonus.** Holders with AGENT/SHARDS in any qualifying Cardano DEX LP at snapshot (14 days pre-launch, commit-reveal) receive a **+3% cMATRA bonus** from the Ecosystem Treasury. Eligible DEXs: Minswap v1/v2, WingRiders, CSWAP, SundaeSwap, MuesliSwap, Splash, VyFinance, SaturnSwap. See Q24a for details.
3. **Team takes the same haircut.** The 15% dilution applies to the Team carve identically (31.3M → 26.6M cMATRA) — fairness signal, same ratio.
4. **Early redeemers** may receive time-limited MOTRA sponsorship or delegated capacity during onboarding; the Network Incentives Reserve is non-circulating until released on a protocol schedule.

### 12. Is the Team taking the same hit?

**Yes — the Team takes the full 15% haircut alongside public holders.** The carve amount drops from ~31.3M cMATRA at v3.3 rates to ~26.6M cMATRA at v5.1 rates, which is the v3.3 carve × 0.85 to the base unit.

Mechanism: the Team's carve is computed as (waived balance) × (published rate). Because the published rate dropped by 15%, the carve drops by 15% too. No special Team-only adjustment was made.

The current waived balances (unchanged from v3.3) are:

* **AGENT:** 29,644,656 (FPS DAO: 9,902 + $TALOS: 29,634,754)
* **SHARDS:** 446,969.70 display units (all at FPS DAO)

Since the creation of both on-chain treasuries, there has been a complete lack of independent DAO proposals — the only proposals submitted were those proposed by the FPS Team itself. These funds are being returned to the team where they can be put to better use with more flexibility. All waived balances are published transparently so any holder can verify them on-chain.

Because these tokens are held in on-chain Agora DAOs and cannot be withdrawn without governance quorum (which has historically been unachievable), the team receives their cMATRA directly at mint time at the same published rates as public holders. The DAO treasuries effectively become dead wallets — the legacy tokens remain locked there permanently. This does not change public rates because the waiver already excluded these balances from the rate denominator.

### 12a. Why doesn't the team surrender through the pool like everyone else?

The AGENT and SHARDS treasury tokens are locked in on-chain Agora DAOs that require governance votes to move funds. Historically, there has not been enough voter participation to pass any proposals. Rather than risk the team allocation being permanently stuck, the equivalent cMATRA is minted directly to the team at the same published rate. This is economically identical to surrendering through the pool.

### 12b. Does the team carve-out change public rates?

No. The waiver already excluded the team's treasury balances from the rate denominator when computing per-unit rates. The carve-out simply delivers the corresponding cMATRA at those same rates and reduces the surrender pool by that amount (~26.6M cMATRA out of 722.5M at v5.1). Public holders redeem at exactly the same rate regardless.

### 12c. Why does the v3.3 doc show a ~10% implicit team softening but v5.1 is a flat 15% cut?

The v3.3 model applied a treasury-waiver framing that the public could read as a partial team softening. The v5.1 rebalance makes this explicit: there is no team-specific softening in v5.1. The Team's carve recomputes at the same rate × 0.85 multiplier applied to all holders. This is cleaner to communicate and easier to audit — one multiplier, no class-specific carve-outs hidden inside the rate table. The Team is taking the haircut on the same line as every other holder.

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

### 15a. What about Flux Point NFTs that aren't on the eligible list, like banners or weapons?

NFTs with direct in-game utility (e.g. TRIB3 banners, weapons) are **not** part of the merger. They retain their original function in-game and would lose that utility if converted to a fungible token. Only the seven assets listed in Q13 are eligible for cMATRA redemption.

### 16. Do I need to use the same wallet that held the asset at the reference snapshot?

No, not for the ordinary public redemption path.

The important thing is that you control the eligible asset when you surrender it during the window and follow the published redemption instructions.

***

## Rates

### 17. Is AGENT 1:1 with cMATRA?

No.

AGENT is **not** 1:1 with cMATRA, and neither are the other legacy assets. The merger uses a weighted redemption model, not a same-supply shortcut.

### 17a. Why do I get less than 1 cMATRA per AGENT if the supply is also 1 billion?

Because this is not a simple 1:1 token rename. Seven separate assets — AGENT, SHARDS, and five NFT collections — are merging into a single token. The 722,500,000 public redemption pool is split across all of them weighted by market value.

The result is a single token with more holders, better decentralization, and consolidated liquidity instead of seven fragmented assets. The value story is the combined ecosystem, not a per-unit supply match.

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

Each asset gets a weighted share of the 722,500,000 public redemption pool based on the published valuation methodology.

Then:

* Fungible rate = asset bucket / final redeemable supply for that asset
* NFT rate = asset bucket / final redeemable NFT count for that collection

### 20a. Can NFT holders manipulate the floor price to game their allocation?

The allocation weights are derived from a **7-day time-weighted average price (TWAP)**, not a single spot price. A 7-day window makes short-term wash trading or floor manipulation expensive and impractical — you would need to sustain artificial volume for an entire week to move the average meaningfully.

Additionally, the TWAP data used for the published reference rates has **already been captured**. The rate weights are locked. Even if someone manipulated a floor price today, it would not change the published allocation.

### 20b. Will rates be recalculated with new market data before launch?

No. The TWAP-derived **weights** (how the pool is split between assets) are locked based on the published reference data. The only thing that may cause a minor rate adjustment before final publication is a change in the **redeemable supply denominator** — specifically, if legacy reward materialization adds units to an asset's redeemable count. The market-price side of the formula is not being re-run.

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

### 24a. How does the LP Loyalty Bonus work?

Holders who have AGENT and/or SHARDS deployed in any qualifying Cardano DEX liquidity pool at the pre-launch snapshot receive a **+3% cMATRA bonus** on top of their base redemption amount. The bonus is funded by the 40M Ecosystem Treasury sub-bucket and is expected to cost ~2–3M cMATRA depending on LP participation.

**Snapshot methodology:**

* **Timing:** 14 days before launch, using a commit-reveal protocol to prevent last-second wash-farming. The commit block is announced publicly; the reveal block uses a future Cardano block hash that is not known at commit time.
* **Data source:** Blockfrost-style indexing of every LP position holding AGENT or SHARDS across the eligible DEX list at the reveal block height.
* **Eligibility rule:** The wallet that controls the LP token at snapshot height is credited with the underlying AGENT/SHARDS held inside that LP position at the same height.
* **Withdrawal still required:** You must unfarm / remove liquidity before the redemption deadline and surrender the underlying tokens normally. The +3% is applied at redemption time to the wallet that was credited at snapshot.

**Eligible DEXs at launch:**

* Minswap v1 and v2
* WingRiders
* CSWAP
* SundaeSwap
* MuesliSwap
* Splash
* VyFinance
* SaturnSwap

The DEX list is fixed at publication time. Positions in DEXs not on this list do not receive the bonus.

This program is designed to reward holders who kept their AGENT and SHARDS productive during the pre-launch period rather than parking them in idle wallets, and to smooth the transition of liquidity from AGENT/SHARDS pairs into the new cMATRA pairs at launch.

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

### 33a. I hold a T1 or T2 ADAM Launch Pass — will I lose access to ADAM if I surrender it?

No. ADAM v2 will no longer require a pass or subscription to access. ADAM v2 will charge per-transaction fees only. The merger is essentially buying out the ADAM passes with cMATRA, since the passes will have no utility under the new ADAM v2 model.

### 34. What if I wait until the last day and the marketplace, DeFi venue, or network is congested?

Do not wait until the last day.

The six-month window is meant to give holders enough time to recover assets from farms, marketplaces, or other venues. Materios cannot guarantee third-party platform responsiveness or network conditions right at the deadline.

***

## After the Window

### 35. What happens after six months?

The public redemption window closes. Unredeemed legacy assets no longer have an open public trade-in path unless an explicit extension or exceptional remedy is formally announced.

### 36. What happens to any unreleased cMATRA from the 722.5M public redemption pool after the deadline?

After the six-month window closes, the administrator withdraws any remaining cMATRA from the surrender pool. No further public surrenders are possible once the deadline has passed. The on-chain validator enforces this — the `ProcessSurrender` spending path is permanently disabled after the deadline, and only the `AdminWithdraw` path is available.

Unclaimed cMATRA returns to the project treasury for future allocation decisions (validator incentives, ecosystem development, or other uses to be determined by governance).

***

## Materios Onboarding

### 37. Do I need to bridge to Materios immediately after redeeming?

No. The intended flow is Cardano-first. Users redeem into cMATRA on Cardano first, then complete address association and later bridge or lock into Materios as the onboarding flow matures.

### 38. Will cMATRA and MATRA be separate supplies?

No. The intended model is one economic supply story. cMATRA is the transitional Cardano-side representation and MATRA is the Materios-native capital token.

***

## Validators

### 39. How are validators expected to be rewarded?

From the **Validator Emissions sub-bucket (115M MATRA, 11.5% of supply)** within the Network Incentives Reserve, not from burned MOTRA fees. In addition, 40% of transaction fees route to a block-author pot during normal operation, so validator income has two sources: the initial emissions schedule plus ongoing fee recycling.

This includes **Cardano SPO delegation rewards**. Materios is integrating the IOG partner-chains cross-validation framework (Minotaur), which allows Cardano stake pool operators to participate in Materios consensus. Delegators to participating SPOs contribute to Materios security through cross-chain validation and receive cMATRA rewards from the Validator Emissions sub-bucket in proportion to their stake.

Attestors are rewarded separately from the **Attestor Emissions sub-bucket (65M MATRA, 6.5%)**, which also receives 30% of transaction fees under the fee-recycling policy. Ecosystem-side activity (grants, LP Loyalty Bonus, integrations) is funded by the **Ecosystem Treasury sub-bucket (40M MATRA, 4%)**, which additionally receives 20% of transaction fees; the remaining 10% of each fee is burned.

### 39a. How do Cardano SPO delegators earn MATRA?

Through cross-validation, Cardano stake pool operators participate in Materios consensus alongside dedicated Materios validators. Delegators to participating pools (starting with **TALOS**) contribute to Materios network security through their delegation, and receive cMATRA from the Validator Emissions sub-bucket in proportion to their stake.

* **Source:** The 115,000,000 MATRA Validator Emissions sub-bucket (11.5% of total supply), inside the 277.5M Network Incentives Reserve.
* **Rationale:** Cross-validation (Minotaur) means SPO delegators directly secure the Materios network. Distributing validator rewards to them is the sub-bucket doing its job.
* **Duration:** An initial distribution period with published start and end dates. Duration and rates are published before the first distribution.
* **Rate:** A fixed cMATRA-per-ADA-per-epoch rate, published in advance.
* **Distribution:** Periodic distribution to delegator wallets based on epoch snapshots.
* **Impact on public pool:** None. Delegation rewards draw exclusively from the Validator Emissions sub-bucket, not the 722,500,000 public redemption pool. Public redemption rates are completely unaffected.
* **ADA rewards:** Delegators continue to receive their normal ADA staking rewards. cMATRA delegation rewards are additional.

### 40. Can unused reserve sub-buckets be repurposed later?

Not casually. The Network Incentives Reserve is not discretionary operating capital. Any later reassignment of clearly surplus reserve should require governance and should be communicated explicitly.

Distributing delegation rewards to SPO delegators who secure Materios through cross-validation (Q39a) is not a repurposing — it is the Validator Emissions sub-bucket fulfilling its stated purpose on Cardano L1 before the full cross-chain bridge is operational.

***

## Market and Valuation

### 41. Do we know the starting market cap of MATRA?

Materios does **not** set a guaranteed opening market cap. Market cap depends on what the market decides to pay once trading starts and on how much supply is actually circulating.

What Materios can publish in advance is the fixed supply model and the reference valuation package used to derive redemption weights.

### 41a. Why launch on SaturnSwap (CLOB) instead of Minswap (AMM)?

Materios chose a **CLOB-primary launch** on SaturnSwap for four reasons:

1. **Capital efficiency.** At the initial liquidity scale (~$250K per pair), a CLOB delivers tighter spreads and deeper effective depth than an AMM pool with the same capital, because capital can be concentrated around the mid-price as resting orders rather than spread along an x·y=k curve.
2. **Native maker-rebate economics.** The Liquidity sub-bucket funds a maker-rebate program (5M MATRA over 24 months, 50% annual decay, ~15 bps rebate on a 30 bps maker fee, 20% cap per market-maker address with two-sided-quote requirements). Maker rebates map cleanly onto CLOB economics; forcing them onto AMM LP rewards creates wrong incentives (farm-and-dump, no spread obligation).
3. **Own infrastructure.** SaturnSwap is part of the same ecosystem Materios is building into (via CIP-68 Liftoff integration), which means the launch venue and the redemption venue share operational context, incident response, and upgrade cadence.
4. **Price discovery.** An order book makes the launch price legible in real time (visible bids and asks) rather than derived from whatever size happens to hit a bonding curve.

**Secondary AMM listings on Minswap and WingRiders** provide retail access for users and aggregators that only interact with AMMs. Arbitrageurs keep prices aligned between the CLOB and the AMMs. These AMM pools are smaller than the CLOB book and are intentionally designed as arbitrage targets, not as the primary price-discovery venue.

Materios considered a Liquidity Bootstrapping Pool (LBP) and decided against it: the offshore-entity and legal overhead ($30–80K legal, 2–3 month timeline) is not worth it at the $270K launch-pair scale.

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

***

**Version:** 7.0 | **Date:** April 19, 2026 | **Status:** Public / governance draft\
**Companion documents:** Litepaper, eligibility rules, fixed rate table, legacy reward reconciliation package, validator incentives spec
