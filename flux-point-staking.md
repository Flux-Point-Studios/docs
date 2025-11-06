---
description: Earn daily AGENT & SHARDS, non‑custodially, by staking from your own wallet.
---

# Flux Point Staking

**Start staking:** [https://fluxpoint.ada-anvil.io/en/](https://fluxpoint.ada-anvil.io/en/)

***

### TL;DR

* **Non‑custodial:** Assets stay locked to **your wallet’s stake key**. You remain the holder while staked.
* **Daily rewards:** We snapshot **00:00 UTC**. Rewards accrue **per day** and can be **claimed anytime**.
* **No lockup cap:** Stake for a week or a year—**no maximum**. Harvest whenever.

***

### How to Stake (3 steps)

1. **Open the dApp** and connect your Cardano wallet.
2. **Select what to stake** (AGENT, SHARDS, and—if boosting SHARDS—eligible NFTs).
   * **Important:** **NFT boosts apply only if the NFTs and SHARDS are staked in the same transaction under the same stake key.**
3. **Submit (2 ADA fee)** and you’re in. Changes made after 00:00 UTC affect the **next** day’s accrual.

> **Per‑tx limit:** Up to **100 assets per transaction**.

***

### What Can I Stake?

**Tokens**

* **$SHARDS** (Cardano policy: `ea153b5d4864af15a1079a94a0e2486d6376fa28aafad272d15b243a`)
* **$AGENT** (Cardano asset: `97bbb7db0baef89caefce61b8107ac74c7a7340166b39d906f174bec54616c6f73`)

**NFTs that boost SHARDS only**

* **Flux Point Team Pass** — +**0.25%** per NFT\
  `0889a2d542897f0c7eefed47d2d809bd8d8ec78881bd4ff9464f683a`
* **Brawl Pass: Enter the Dragon** — +**0.15%** per NFT\
  `d3a197c4814054623432c882c60e6a81e8f3b94158033432529a02d2`
* **SE Brawlers** — +**0.10%** per NFT\
  `25c75bbf105310685d51cd3adbdd50b72fdbd99be2cc3757dde7eafc`

> Boosts are **additive per NFT** and only apply when **bundled with SHARDS** in the same stake.

***

### How Rewards Work

#### 1) Daily Snapshot & Buckets

* We take a **snapshot at 00:00 UTC** every day.
* That day’s **reward buckets** are split **pro‑rata** across **only** the stake positions present at the snapshot.

> Buckets are **dynamic**: they move up/down based on ongoing streams into the treasury and total stake. You can track the treasury here: [https://pool.pm/addr1q9h0s6lghyjszcmk3zwg2pyy22r2wjt9mm0m6rxlxtl93snwlp473wf9q93hdzyus5zgg55x5aykthklh5xd7vh7trpqc08pfv](https://pool.pm/addr1q9h0s6lghyjszcmk3zwg2pyy22r2wjt9mm0m6rxlxtl93snwlp473wf9q93hdzyus5zgg55x5aykthklh5xd7vh7trpqc08pfv)

**Example campaign math (illustrative):**

* **AGENT:** 750,000 over 60 days ⇒ **12,500/day**
* **SHARDS:** 50,000 over 60 days ⇒ **\~833/day**

_(Actual daily amounts can vary with treasury inflows and total staked.)_

***

#### 2) AGENT Distribution (no NFTs involved)

* **Per‑token/day rate** = `12,500 / (total AGENT staked that day)`.
* **Your AGENT** = `12,500 * (your AGENT / total AGENT)`.
* **Payout rounding:** **AGENT has 0 decimals.** We use **largest‑remainder** rounding so the day sums **exactly** to 12,500.

***

#### 3) SHARDS Distribution (with NFT multipliers)

* **Daily bucket** ≈ `833 SHARDS` (example).
* **Effective weight per user** = `SHARDS_staked × (1 + boost)` where:
  * **Team Pass:** +0.25% per NFT
  * **Brawl Pass: Enter the Dragon:** +0.15% per NFT
  * **SE Brawlers:** +0.10% per NFT
* **Your SHARDS** = `bucket × (your_effective_weight / total_effective_weight)`.

**Rounding:**

* If **SHARDS = 0‑decimals**, we floor per user and assign leftovers via **largest‑remainder** so the day totals the bucket.
* If **SHARDS supports decimals**, we can emit fractional amounts (e.g., 50,000/60 = **833.3333/day**).

**Quick example (SHARDS):**\
Stake **100,000 SHARDS** + **2 Team Pass** + **1 Brawl Pass** + **5 SE Brawlers**\
Boost = `2×0.25% + 1×0.15% + 5×0.10% = 1.15%`\
Effective weight = `100,000 × 1.0115 = 101,150`\
Daily reward = `833 × (101,150 / total_effective_weight)` → rounded as above.

***

### Fees, Timing, and Controls

* **Fee:** **2 ADA** at **stake**/**claim**.
* **Claim/harvest anytime.** Rewards accrue **daily**, not block‑by‑block.
* **Unstake anytime.** If you restake, the **next** snapshot determines accrual.
* **No maximum duration.** You can leave it staked for a year and harvest later.

***

### Revenue Streams Powering Rewards

* **SHARDS stakers** receive **33%** of revenue from **gaming products** (e.g., TRIB3) and **tools** (e.g., Cardano UE SDK).
  * Games: [https://fluxpointstudios.com/games/trib3](https://fluxpointstudios.com/games/trib3)
  * Tools: [https://fluxpointstudios.com/cardano-ue-sdk](https://fluxpointstudios.com/cardano-ue-sdk)
* **AGENT stakers** receive:
  * **28%** of revenue from **Proof‑of‑Inference (PoI)** jobs — [https://docs.fluxpointstudios.com/proof-of-inference-litepaper](https://docs.fluxpointstudios.com/proof-of-inference-litepaper)
  * **25%** of **AGENT Babel fees** returned via **ADAM** txs — [https://fluxpointstudios.com/adam](https://fluxpointstudios.com/adam)
  * **25%** of **FPS revenue from SaturnSwap.io** as staking rewards.

> Net effect: daily buckets adjust as revenue streams flow into the treasury and as stake totals change.

***

### FAQ (straight answers)

* **Do I need NFTs for AGENT rewards?**\
  No. NFTs **do not** impact AGENT distribution.
* **Do NFTs need to be staked with SHARDS to boost?**\
  Yes. **Only** if the **NFTs and SHARDS** are part of the **same stake** (same tx, same stake key).
* **What if I staked NFTs first, then SHARDS later in a separate tx?**\
  Boost **won’t** apply. **Unstake and restake together** (within the 100‑asset tx limit).
* **When do changes count?**\
  At the **next 00:00 UTC** snapshot.
* **Is this custodial?**\
  No. Staking is **tied to your wallet’s stake key**.
* **Can I add to an existing stake?**\
  No. You will need to claim and then re-stake to add additional assets to a stake.
* **Is there a minimum amount I need to stake to start earning rewards?**\
  Yes. Rewards and the amount needed changes daily depending on the amount of token rewards in the treasury, amount staked, etc. You can see the updated amount at [https://fluxpointstudios.com/tokens](https://fluxpointstudios.com/tokens) in the header widget.

***

### Copy/Paste Policy IDs (for verification)

```
SHARDS policy: ea153b5d4864af15a1079a94a0e2486d6376fa28aafad272d15b243a
AGENT asset:  97bbb7db0baef89caefce61b8107ac74c7a7340166b39d906f174bec54616c6f73

NFTs (boost SHARDS):
Team Pass:     0889a2d542897f0c7eefed47d2d809bd8d8ec78881bd4ff9464f683a
Brawl Pass:    d3a197c4814054623432c882c60e6a81e8f3b94158033432529a02d2
SE Brawlers:   25c75bbf105310685d51cd3adbdd50b72fdbd99be2cc3757dde7eafc
```

***

#### Plain‑English Fine Print

* Rewards accrue **once per day** at snapshot, based on what’s staked **at that moment**.
* **Integer tokens** use **largest‑remainder** rounding so the daily sum matches the bucket precisely.
* Buckets are **variable**: treasury inflows and total stake change the **per‑user/day** amounts.

***
