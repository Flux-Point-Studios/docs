---
description: >-
  Seven legacy Cardano assets are consolidating into cMATRA — the Cardano-side
  capital token for Materios. This page covers the eligibility rules and
  redemption model.
---

# cMATRA Token Merger

Seven legacy Cardano assets are being consolidated into **cMATRA**, the Cardano-side transitional token for [Materios](https://materios.network). cMATRA is intended to converge into **MATRA** as the native capital token of the Materios network.

***

## Quick Facts

| Item                    | Value                                          |
| ----------------------- | ---------------------------------------------- |
| Output token            | cMATRA / MATRA                                 |
| Max supply              | 1,000,000,000                                  |
| Decimals                | 6                                              |
| Validator reserve       | 150,000,000 (15%)                              |
| Public redemption pool  | 850,000,000 (85%)                              |
| Public window           | 6 months from launch                           |
| Public model            | Surrender legacy assets, receive cMATRA        |
| Snapshot role           | Audit baseline, not normal fungible entitlement |

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

The merger uses a fixed supply model with a validator reserve carved out at genesis.

* The **15% validator reserve** is non-circulating at launch and exists to fund Materios validators over time.
* The **85% public redemption pool** is the only pool used for ordinary public redemptions.
* **Team treasury softening** is handled through explicit waiver and disclosure, not by pretending the reserve does not exist.
* **Team carve-out:** Team treasury balances locked in on-chain DAOs are waived from the public rate denominator and receive cMATRA directly at mint time at the same published rate. This reduces the surrender pool by the carve amount (~31.3M cMATRA) but does not change public rates. See [FAQ](faq.md) for details.

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

Each asset receives a weighted share of the **850,000,000 public redemption pool** using the published valuation methodology:

* **Fungible tokens:** 7-day TWAP from top-pool market data
* **NFT collections:** 7-day TWAP of collection floor prices
* **Final allocation:** Deterministic weighted split of the public redemption pool

### Rate formulas

For fungible assets:

```
asset_bucket = asset_weight × 850,000,000
redemption_rate = asset_bucket / final_redeemable_supply
```

For NFTs:

```
asset_bucket = asset_weight × 850,000,000
per_nft_redemption = asset_bucket / final_redeemable_nft_count
```

### Reference rate table (draft — March 12, 2026)

The following rates are based on 7-day TWAP data from March 12, 2026 and the current team waiver balances. **These are reference rates, not final.** Final rates will be recalculated with fresh TWAP data closer to launch.

| Asset                         | Bucket (cMATRA)      | Redeemable Supply  | Rate per Base Unit | Rate per Display Unit |
| ----------------------------- | -------------------: | -----------------: | -----------------: | --------------------: |
| AGENT                         | 528,433,002          | 970,355,344        | ~0.5446            | ~0.5446 per AGENT     |
| SHARDS                        | 90,504,944           | 2,641,564.75       | ~0.0000342         | ~34.2 per SHARDS      |
| Flux Point Team Pass          | 32,679,426           | 401 NFTs           | —                  | ~81,495 per NFT       |
| SE Brawlers                   | 10,048,467           | 242 NFTs           | —                  | ~41,523 per NFT       |
| Brawl Pass: Enter the Dragon  | 5,813,163            | 44 NFTs            | —                  | ~132,117 per NFT      |
| T1 ADAM Launch Pass           | 161,471,298          | 43 NFTs            | —                  | ~3,755,146 per NFT    |
| T2 ADAM Launch Pass           | 21,049,701           | 95 NFTs            | —                  | ~221,576 per NFT      |

*All values in display units (6 decimal places). AGENT has 0 decimals so base = display. SHARDS has 6 decimals (1 SHARDS = 1,000,000 base units). Buckets sum to exactly 850,000,000 cMATRA.*

### Team carve-out (from public pool)

The following amounts are minted directly to the team at the same published rates. The surrender pool is funded with the remainder.

| Treasury | Asset | Waived Balance | Rate per Display Unit | cMATRA Received |
| -------- | ----- | -------------: | --------------------: | --------------: |
| FPS DAO  | AGENT | 29,644,656     | ~0.5446 per AGENT     | ~16,143,768     |
| FPS DAO  | SHARDS | 446,969.70    | ~34.2 per SHARDS      | ~15,196,970     |
| **Total** | | | | **~31,340,738 cMATRA** |

**Surrender pool after carve: ~818,659,262 cMATRA** (96.3% of public pool).

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
* After the deadline, the normal public surrender path is closed unless an explicit extension or exceptional remedy is formally announced.
* Any unreleased portion of the public pool follows the **published post-deadline policy** in the final launch package.

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

**Version:** 3.2 | **Date:** March 12, 2026 | **Status:** Public / governance draft
