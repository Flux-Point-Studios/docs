---
description: >-
  Seven legacy Cardano assets are consolidating into cMATRA — the Cardano-side
  capital token for Materios. This page covers the eligibility rules and
  redemption model.
---

# cMATRA Token Merger

Seven legacy Cardano assets are being consolidated into **cMATRA**, the Cardano-side transitional token for [Materios](https://materios.fluxpointstudios.com). cMATRA is intended to converge into **MATRA** as the native capital token of the Materios network.

***

## Quick Facts

| Item                         | Value                                          |
| ---------------------------- | ---------------------------------------------- |
| Output token                 | cMATRA / MATRA                                 |
| Max supply                   | 1,000,000,000                                  |
| Decimals                     | 6                                              |
| Network Incentives Reserve   | 277,500,000 (27.75%)                           |
| Public Redemption Pool       | 722,500,000 (72.25%)                           |
| Public window                | 6 months from launch                           |
| Public model                 | Surrender legacy assets, receive cMATRA        |
| Snapshot role                | Audit baseline, not normal fungible entitlement |

***

## Included Assets

### Fungible Tokens

| Token  | Policy ID                                                    | Decimals |
| ------ | ------------------------------------------------------------ | -------: |
| AGENT  | `97bbb7db0baef89caefce61b8107ac74c7a7340166b39d906f174bec`   |        0 |
| SHARDS | `ea153b5d4864af15a1079a94a0e2486d6376fa28aafad272d15b243a`   |        6 |

### NFT Collections

| Collection                    | Policy ID                                                    | Redeemable Count |
| ----------------------------- | ------------------------------------------------------------ | ---------------: |
| Flux Point Team Pass          | `0889a2d542897f0c7eefed47d2d809bd8d8ec78881bd4ff9464f683a`   |              401 |
| SE Brawlers                   | `25c75bbf105310685d51cd3adbdd50b72fdbd99be2cc3757dde7eafc`   |              242 |
| Brawl Pass: Enter the Dragon  | `d3a197c4814054623432c882c60e6a81e8f3b94158033432529a02d2`   |               44 |
| T1 ADAM Launch Pass           | `b46891456b77dbc77c16090fd92a37f087f9a68e953c56b00a20332f`   |               43 |
| T2 ADAM Launch Pass           | `06a64965c0ac1144a72a6ddfcb23aa9d4d7742a5b20ddd5cfb1164b9`   |               95 |

***

## Supply Model

The merger uses a fixed supply model with a **Network Incentives Reserve** carved out at genesis, split into five purpose-built sub-buckets.

* The **27.75% Network Incentives Reserve (277.5M cMATRA)** is non-circulating at launch and funds long-term network security, ecosystem growth, strategic partnerships, and launch liquidity. It is not a later inflation switch — it is part of the genesis supply plan.
* The **72.25% Public Redemption Pool (722.5M cMATRA)** is the only pool used for ordinary public redemptions. Network Incentives sub-bucket distributions do not draw from this pool.
* **Fee-recycling** supplements the reserves once the network is live: transaction fees are split 40% to a block-author pot, 30% to the attestor reserve, 20% to the Ecosystem Treasury, and 10% burned. This means validator and attestor incentives have multiple sources — genesis reserve plus ongoing fee flow.
* **Team treasury softening** is handled through explicit waiver and disclosure. Team treasury balances locked in on-chain DAOs are waived from the public rate denominator and receive cMATRA directly at mint time at the same published rate. This reduces the surrender pool by the carve amount (~26.6M cMATRA at v5.1 rates) but does not change public rates. See [FAQ](faq.md) for details.

### Network Incentives Reserve sub-buckets

| Sub-bucket                    | MATRA      | % of 1B  | Purpose                                                         |
| ----------------------------- | ---------: | -------: | --------------------------------------------------------------- |
| Validator Emissions           | 115,000,000 |  11.5%  | Block-production rewards schedule; Cardano SPO cross-validation |
| Attestor Emissions            |  65,000,000 |   6.5%  | Threshold-attestation rewards + bond/slash economics            |
| Ecosystem Treasury            |  40,000,000 |   4.0%  | Grants, dev funding, governance-directed programs               |
| Strategic Allocation          |  30,000,000 |   3.0%  | Strategic/institutional partners (12mo cliff + 36mo linear)     |
| Liquidity (total)             |  27,500,000 |  2.75%  | Bridge peg reserve, Protocol-Owned Liquidity, maker rebates     |
| **Total reserve**             | **277,500,000** | **27.75%** |                                                        |

The Liquidity bucket breaks down as 5M for bridge peg reserve, 17.5M for Protocol-Owned DEX Liquidity (POL), and 5M for maker rebates on the CLOB launch venue.

***

## Launch Venue

Materios is launching cMATRA on **[SaturnSwap](https://saturnswap.io)**, a Central Limit Order Book (CLOB) DEX on Cardano, as the primary venue.

* **Primary venue — SaturnSwap CLOB:** cMATRA/ADA and cMATRA/USDM pairs, seeded with resting orders at 25/50/100/200/400 bps from mid. Target book depth: $250K within 2% of mid at launch. CLOB gives better capital efficiency and price discovery than AMMs for a launch of this size; native maker-rebate economics are also a better fit than fixed-schedule LP mining.
* **Secondary venues — AMMs:** [Minswap](https://minswap.org) ($75–100K depth) and [WingRiders](https://www.wingriders.com) ($25–50K depth) for users and routers that only interact with AMMs. These act as arbitrage targets against the CLOB.

A CLOB-primary launch is enabled by SaturnSwap's CIP-68 Liftoff integration, which lets new tokens list with full order-book functionality on day one rather than waiting for AMM liquidity to deepen. The maker-rebate program funded by the Liquidity sub-bucket (5M MATRA over 24 months, 50% annual decay, paid in MATRA at ~15 bps of a 30 bps maker fee, 20% cap per market-maker address with two-sided-quote requirements) keeps spread tight from the first block.

***

## Liquidity Programs

### LP Loyalty Bonus

Holders who have AGENT and/or SHARDS deployed in any qualifying Cardano DEX liquidity pool at the pre-launch snapshot receive a **+3% cMATRA bonus** on top of their base redemption amount.

* **Snapshot timing:** 14 days before launch, using a commit-reveal protocol to prevent last-second wash-farming.
* **Eligible DEXs:** Minswap v1 and v2, WingRiders, CSWAP, SundaeSwap, MuesliSwap, Splash, VyFinance, and SaturnSwap.
* **Coverage:** The bonus is funded from the Ecosystem Treasury sub-bucket; expected cost ~2–3M cMATRA depending on LP participation at snapshot.
* **Eligibility rule:** The bonus is credited to the wallet that controls the LP position at snapshot time, applied to the underlying AGENT and SHARDS that the LP token represents when withdrawn and redeemed.

This program rewards holders who kept their AGENT and SHARDS productive during the pre-launch period rather than parking them in idle wallets.

***

## Strategic Allocation

The 30M MATRA Strategic Allocation sub-bucket (3% of supply) is reserved for institutional partners who commit capital to the network ahead of the public launch.

* **Vesting:** 12-month cliff followed by 36-month linear vesting. No pre-cliff liquidity.
* **Purpose:** The strategic round is expected to fund Protocol-Owned Liquidity seeding, security audits, and runway for team expansion beyond the redemption window.
* **Transparency:** All strategic allocations will be disclosed on-chain at mint time and the vesting contracts will be independently verifiable via the Materios explorer.

This bucket is held by Materios until strategic partners are confirmed. None of these tokens enter circulation during the 12-month cliff regardless of partner timing.

***

## Post-Reconciliation Dilution Rationale

The v5.1 allocation represents a **15% downward adjustment** from the v3.3 (March 2026) reference rates: public holders now receive approximately 85% of what the previous rate table published, and the Team carve-out takes exactly the same 15% haircut.

This falls squarely within the governance-approved correction clause stated in the "Window mechanics" section above:

> *"The rate table remains fixed for the full window unless a documented governance-approved correction is required for a launch bug or reconciliation error."*

The v3.3 rates assumed a pure 85% public / 15% reserve split that did not account for attestor emissions (already committed in the runtime code as a separate 50M reserve), left no room for ecosystem treasury or strategic partners, and produced rate-table totals that were mathematically inconsistent with the committed reserve architecture. The v5.1 rebalance reconciles published rates with the runtime's actual reserve structure, builds in strategic and liquidity sub-buckets that enable a real launch, and applies the haircut evenly — including to the Team carve (31.3M → 26.6M cMATRA) — so no holder class is disadvantaged relative to any other.

Final fixed rates are republished below as of the v5.1 publication date.

***

## Public Redemption Model

The merger uses a **surrender-and-redeem model** for normal public holders.

### Core rule

A public holder redeems by **actually controlling and surrendering an eligible legacy asset during the redemption window**.

* Redemption follows **present control plus surrender**, not snapshot ownership alone.
* A holder who acquires an eligible legacy asset before the deadline may redeem it.
* A holder who sells an eligible legacy asset before redeeming no longer controls the redeemable unit.
* The public process is intended to feel like a real trade-in, not a snapshot-only entitlement airdrop.

### Window mechanics

* The standard public window is **6 months** from the published launch date.
* A **fixed rate table** is published before the window opens.
* The rate table remains fixed for the full window unless a documented governance-approved correction is required for a launch bug or reconciliation error.
* Holders can redeem in **multiple transactions** during the window.

***

## What the Reference Snapshot Still Does

A published reference snapshot still exists, but its role has changed.

**The snapshot is used for:**

* Auditability and public reconciliation
* Publication of Team treasury waivers and other disclosed non-public balances
* Legacy reward reconciliation before launch
* NFT collection inventory review and CIP-68 filtering
* Sanity checks for the final launch package

**The snapshot is NOT the normal fungible entitlement rule.** For AGENT and SHARDS, the snapshot does not give ordinary public holders their redemption right by itself.

***

## Fungible Asset Rules

### Redeemable units

The redeemable fungible assets are AGENT, SHARDS, and any valid legacy rewards that Materios explicitly honors and **materializes into actual redeemable units before launch**.

### Current control matters

For ordinary public redemption, redeemability follows **current control of the fungible unit at redemption time**. Materios is not relying on provenance tracking of ordinary fungible units once they are circulating.

### Explicit exclusions

The following categories do not compete with the public redemption pool:

1. **Disclosed Team treasury balances** that are explicitly waived from public redemption.
2. **Balances already burned, quarantined, or permanently immobilized** under the published launch controls.
3. **Any other specifically disclosed non-public balances** that the final launch package excludes by explicit publication.

#### Disclosed Team treasury addresses

| Address                                                            | Label                  | AGENT Waived   | SHARDS Waived     |
| ------------------------------------------------------------------ | ---------------------- | -------------- | ----------------- |
| `addr1w9u9mw864yszpqk7374wtwtwludpa0rc9dmante78c7c9sqqdlyy9`      | $SHARDS DAO Treasury   | 9,902          | 446,969.70        |
| `addr1wx84ytuumke8gxex0l8par4852ey7l4eq6h325rnez0yluc56x0dj`      | $AGENT DAO Treasury    | 29,634,754     | 0                 |
| **Total waived**                                                   |                        | **29,644,656** | **446,969.70**    |

#### Why treasury balances are waived

Since the creation of both on-chain treasuries, there has been a complete lack of independent DAO proposals — the only proposals submitted were those proposed by the FPS Team itself. Given this, it makes the most sense for these funds to be returned to the team where they can be put to better use with more flexibility, rather than sitting idle in governance structures that have seen no community participation. The same rationale applies to the $AGENT DAO treasury.

All waived balances are published transparently so any holder can verify them on-chain.

### LP farms, DEX positions, and DeFi custody

If AGENT or SHARDS are in a Minswap or WingRiders liquidity pool, a farm, a lending protocol, or another script-controlled DeFi position, the holder must first **unfarm / withdraw / remove liquidity** so the underlying tokens return to a wallet they control.

* **LP tokens do not redeem.** They are receipts for positions, not merger assets.
* You redeem the **underlying AGENT and/or SHARDS** once they are back in your wallet.
* If impermanent loss or trading activity changed the amount, you redeem **what you actually withdraw and control**.

### Custodial and exchange balances

Assets held on a centralized exchange or by another custodian are not directly redeemable unless the holder can move them into self-custody during the window.

***

## NFT Rules

Eligible NFTs redeem on an asset-by-asset basis.

* Only assets with **on-chain quantity exactly 1** count as NFTs for merger eligibility.
* For CIP-68 collections, only the **user token** counts. The reference token does not redeem.
* The redeeming holder must control the **exact NFT asset** during the redemption window.
* If an NFT is listed on a marketplace or held by a script, it must first be withdrawn back to a wallet the holder controls.

***

## Legacy Reward Materialization

If a user has valid legacy staking rewards or dashboard-tracked rewards that Materios commits to honor, those rewards are intended to be:

1. Reconciled before launch.
2. Credited or minted into actual redeemable AGENT or SHARDS units before the window opens.
3. Redeemed through the same public surrender path as any other eligible unit.

No separate manual side-claim should be necessary for ordinary honored legacy rewards.

***

## Rate Methodology

### Asset weighting

Each asset receives a weighted share of the **722,500,000 public redemption pool** using the published valuation methodology:

* **Fungible tokens:** 7-day TWAP from top-pool market data
* **NFT collections:** 7-day TWAP of collection floor prices
* **Final allocation:** Deterministic weighted split of the public redemption pool

### Rate formulas

For fungible assets:

```
asset_bucket = asset_weight × 722,500,000
redemption_rate = asset_bucket / final_redeemable_supply
```

For NFTs:

```
asset_bucket = asset_weight × 722,500,000
per_nft_redemption = asset_bucket / final_redeemable_nft_count
```

### What you get (reference rates — April 19, 2026)

The following table shows how much cMATRA you receive for each legacy asset you surrender at the v5.1 rate table. These rates are derived by applying the post-reconciliation 0.85 multiplier to the locked TWAP weights, so the ordering and relative value between assets is unchanged — only the absolute cMATRA output is reduced. Final fixed rates are published after legacy reward materialization is reconciled.

| You surrender                  | You receive (cMATRA)         |
| ------------------------------ | ---------------------------: |
| 1 AGENT                        | ~0.4629                      |
| 1 SHARDS                       | ~28.9                        |
| 1 Flux Point Team Pass         | ~69,271                      |
| 1 SE Brawler                   | ~35,294                      |
| 1 Brawl Pass: Enter the Dragon | ~112,300                     |
| 1 T1 ADAM Launch Pass          | ~3,191,874                   |
| 1 T2 ADAM Launch Pass          | ~188,339                     |

The SHARDS rate is computed from an exact rational ratio (`76,929,202,239,379 / 2,641,564,750,001` in base units) and rendered here to four significant figures. The on-chain mint uses the exact rational to guarantee the bucket sum is 722.5M to the base unit. All rates are fixed for the full 6-month redemption window once published and you can redeem in multiple transactions.

### Detailed rate breakdown

The rates above are derived from a 7-day TWAP valuation of each asset's market price, weighted against the 722,500,000 cMATRA public redemption pool.

| Asset                         | Bucket (cMATRA)      | Redeemable Supply  | Rate per Base Unit | Rate per Display Unit |
| ----------------------------- | -------------------: | -----------------: | -----------------: | --------------------: |
| AGENT                         | 449,168,051.406      | 970,355,344        | 462,890            | ~0.4629 per AGENT     |
| SHARDS                        | 76,929,202.239       | 2,641,564.750001   | 29                 | ~28.9 per SHARDS      |
| Flux Point Team Pass          | 27,777,511.969       | 401 NFTs           | —                  | ~69,271 per NFT       |
| SE Brawlers                   | 8,541,197.187        | 242 NFTs           | —                  | ~35,294 per NFT       |
| Brawl Pass: Enter the Dragon  | 4,941,188.455        | 44 NFTs            | —                  | ~112,300 per NFT      |
| T1 ADAM Launch Pass           | 137,250,603.008      | 43 NFTs            | —                  | ~3,191,874 per NFT    |
| T2 ADAM Launch Pass           | 17,892,245.735       | 95 NFTs            | —                  | ~188,339 per NFT      |

*All values in display units (6 decimal places). AGENT has 0 decimals so base = display. SHARDS has 6 decimals (1 SHARDS = 1,000,000 base units). Buckets sum to exactly 722,500,000 cMATRA (verified against the canonical `rate_table_cmatra.json` in flux-merger `audit_pack/2026-04-19/`).*

### Team carve-out (from public pool)

The following amounts are minted directly to the team at the same published rates. The surrender pool is funded with the remainder. The Team takes exactly the same 15% haircut as public holders — the carve recomputes automatically at the new rates.

| Treasury | Asset | Waived Balance | Rate per Display Unit | cMATRA Received |
| -------- | ----- | -------------: | --------------------: | --------------: |
| FPS DAO  | AGENT | 29,644,656     | ~0.4629 per AGENT     | ~13,722,223     |
| FPS DAO  | SHARDS | 446,969.70    | ~28.9 per SHARDS      | ~13,016,914     |
| **Total** | | | | **~26,739,137 cMATRA** |

**Surrender pool after carve: ~695,760,863 cMATRA** (96.3% of public pool).

### Rate publication conditions

Final fixed redemption rates are published only after:

* Legacy reward materialization is complete
* Final Team treasury waivers are published
* The launch reconciliation package is signed off

***

## Post-Surrender Treatment

Surrendered legacy assets are **not** intended to remain in free circulation. Where possible they are burned, permanently locked, quarantined, or otherwise removed from circulation under the published merger controls.

***

## Deadline and After-Window Handling

* The ordinary public window closes **6 months** after launch.
* After the deadline, the normal public surrender path is permanently closed. The on-chain validator enforces this — the `ProcessSurrender` spending path is disabled after the deadline.
* Any unreleased cMATRA remaining in the pool is withdrawn by the administrator and returned to the project treasury for future allocation decisions (validator incentives, ecosystem development, or other uses to be determined by governance).
* No extension or exceptional late-surrender mechanism is planned. Holders are encouraged to redeem well before the deadline.

***

## Authoritative Launch Materials

If earlier reports, Discord posts, or snapshot-era drafts conflict with this document, the authoritative launch package is:

1. The final litepaper
2. The final eligibility rules
3. The final FAQ
4. The final fixed rate table
5. The legacy reward reconciliation / materialization package
6. The final Team waiver publication
7. The final redemption instructions

***

**Version:** 4.0 | **Date:** April 19, 2026 | **Status:** Public / governance draft
