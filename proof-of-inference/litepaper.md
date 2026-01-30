---
description: Updated October 14, 2025
---

# Proof-of-Inference Litepaper

**The $AGENT Standard for AI: Babel‑Fee Inference & On‑Chain Proof**

***

### 0) TL;DR

* **Mission:** Make AI usage **verifiable** and make it **pay** operators and stakers—natively in **$AGENT**.
* **Core loop:** Stake/bond **$AGENT → run inference jobs → every fee paid in $AGENT → epoch‑roll payouts to stakers → slash on misbehavior**.
* **Receipts:** Each job emits a **signed PoI receipt** (model ID, prompt hash, seed/params, rolling hash). A compact fingerprint is **anchored on Cardano** for audit.
* **Zero‑ADA UX:** With **Babel‑fees**, users sign with **$AGENT only**. Protocol handles any ADA behind the scenes.
* **Economics:** **70% operator / 28% stakers / 1% DAO / 1% FeeTank**—all sourced from **$AGENT fees** (DAO + FeeTank slices are swapped to ADA as needed).
* **Defensible:** Deterministic runs + on‑chain anchors + slashable bonds. No “trust us.”

***

### 1) Investor brief — why this matters

* **The shift:** Big labs are pushing **Model‑as‑an‑App**—closed endpoints, minimal visibility, usage rents.
* **The gap:** Enterprises, developers, and regulators need **verifiable AI**: who ran what, with which model, from which inputs, producing which outputs—with a proof they can check later.
* **Our answer:** **PoI on Cardano** with **$AGENT as native gas**. Jobs pay in $AGENT, receipts are cryptographically anchored on‑chain, and rewards flow **directly** to operators and stakers.

**Verifiable AI > Walled Gardens.** We replaced “trust” with **proof** and aligned incentives with real usage.

***

### 2) Problem

Centralized AI today is a black box:

* No durable receipts; limited audit trails
* Weak recourse for tampering or hallucination
* Poor economics for decentralized providers (can’t finance GPUs on “maybe” yields)

Result: compliance risk for institutions and a starvation diet for independent compute networks.

***

### 3) Solution — PoI on Cardano with **$AGENT** as gas

We run **deterministic inference** (fixed seed/params, rolling hashes), emit a **signed receipt**, and **anchor** a compact fingerprint on‑chain under a standard metadata label. Fees and gas are paid **in $AGENT** via Cardano’s **Babel‑fee** mechanism; users never handle ADA.

**What’s different**

* **Auditable by default:** Every job leaves a verifiable proof trail.
* **Usage‑aligned:** Fees in **$AGENT** go straight to operators and stakers.
* **Credible security:** Operators **stake/bond** $AGENT; misbehavior is **slashable** on‑chain.
* **Clean UX:** **$AGENT‑only** signing; protocol manages ADA where needed (treasury & ops).

***

### 4) What we just demonstrated (preview testnet)

* **Decentralized inference** on a Petals‑based BLOOM‑560M swarm _(runner endpoint withheld)_
* Generated a **signed PoI receipt** (prompt\_hash, seed, params, transcript.rolling\_hash, model identity)
* **Anchored** the receipt fingerprint on Cardano (immutable proof)
* **Escrow initiation** captured on‑chain (separate tx)

**Network:** Cardano **preview** testnet\
**Agent address:**

```
addr_test1qpjzvg6qqefx4c8eqsez4fnsxfdmcjtz4ku9f0fgju6ymc7ehhk8tdd2j0fqtn7975krepy8a7l8cepk5gyzyq7g6l6s4aetl5
```

**Job:** `petals_1758832859921` (seed=53)\
**Receipt (signed):** `receipts/petals_1758832859921.poi.receipt.json`

**Key on‑chain transactions (preview)**

* **Escrow initiate** — label **2222**, `action:"initiate"`\
  `3a7b52a2852b05fba71301716c11dd486de295bd68d33013d28a3b1069bbe325`
* **PoI anchor** — label **2222**, `action:"anchor"`\
  `1a697155ec50babafde25d5908c19a6b0c80c068476ef6ff84d0048e3fdef344`\
  Metadata includes\
  `jobId: petals_1758832859921`\
  `resultHash: 631166715f80b93133221ca4e62df6c03fdcca693249485676929452b5bf82aa`\
  `receiptHash: 14fbdfe780ce6223f549ea514db3493a5b3ea4ceb0cb0f17a09880bd6e33b3e2`

**Interpretation**\
`resultHash` = receipt’s `transcript.rolling_hash` (binds generated tokens).\
`receiptHash` = SHA‑256 of the **signature‑stripped** receipt JSON (enables universal recomputation).

***

### 5) Protocol architecture

| Layer                    | Component                 | Role                                                               | Notes                                 |
| ------------------------ | ------------------------- | ------------------------------------------------------------------ | ------------------------------------- |
| **On‑chain (Plutus V3)** | **InferencePool**         | Holds bonded **$AGENT**; slash on misbehavior                      | Deterministic policy parameters       |
|                          | **JobEscrow**             | Escrows **$AGENT** fees; enforces proof & **70/28/1/1** split      | Label **2222** events                 |
|                          | **StreamDistributor**     | Batches & pays staker rewards **once per epoch** in $AGENT         | Gas‑efficient payouts                 |
|                          | **PoI Anchor (metadata)** | Stores compact job fingerprint (jobId, resultHash, receiptHash)    | Standardized schema (label 2222)      |
|                          | **FeeTank (Aquarium)**    | Babel‑fee provider & ADA sink for UTxO/ops                         | Auto‑topped‑up (1% slice)             |
| **Off‑chain**            | Dispatcher                | Queues jobs; builds tx metadata; quotes model/time/VRAM            | Signs $AGENT‑only txs                 |
|                          | Verifier / Oracle         | Validates result hash vs receipt; price oracle (Charli3)           | Multi‑sig capable                     |
|                          | Epoch‑Roll Aggregator     | Writes one payout per staker per epoch                             | Minimizes chain load                  |
| **Liquidity & pricing**  | External venues           | Provide $AGENT liquidity and small ADA conversions (for 1% slices) | Use conservative oracle‑guided quotes |

> We no longer rely on, operate, or reference ownership of any DEX. Liquidity access is **venue‑agnostic** and uses conservative quoting plus oracle checks.

***

### 6) Economics & incentives (no buy‑backs—$AGENT is the fee token)

**Fee split (all sourced from $AGENT job fees)**

| Share | Receiver       | Purpose                                       |
| ----: | -------------- | --------------------------------------------- |
|   70% | Pool operator  | Energy / rental / API                         |
|   28% | Stakers        | Usage‑driven rewards (epoch‑rolled in $AGENT) |
|    1% | DAO Treasury   | Swapped to ADA for audits & grants            |
|    1% | FeeTank top‑up | Swapped to ADA to keep Babel/ops solvent      |

* **Staking flywheel:** More inference → more **$AGENT** fees → periodic distributions to stakers → stronger security and liquidity.
* **Operator alignment:** Uptime + correctness = more jobs and revenue; misreports risk **slashing**.
* **No emissions:** **Max supply 1B $AGENT** (fixed). **No new issuance.** Utility demand only.
* **Governance:** Proposals via **Clarity.vote / Agora DAO**. Voting power = **bonded $AGENT**, snapshotted once per epoch.

***

### 7) Full flow

1. User submits job → **pays entirely in $AGENT** (Babel‑fees cover ADA).
2. Runner executes deterministically (seed/params fixed).
3. Runner emits a **signed PoI receipt** (model ID, prompt hash, seed/params, transcript rolling hash).
4. Protocol posts a compact **on‑chain anchor** (label **2222**).
5. **JobEscrow** applies the **70/28/1/1** policy; staker rewards **roll once per epoch**.
6. Fraud proof or SLA miss → **slash** the operator’s bond.

***

### 8) Token utility

* **Bonding:** Pools stake/bond **$AGENT** sized to hardware tier.
* **Fee token:** All jobs quote and settle in **$AGENT**.
* **Governance:** Bonded **$AGENT** = voting power (epoch‑snapshot).

***

### 9) Treasury note

DAO treasury already lives on **Clarity.vote (Agora UTxO)**. The protocol routes **1%** of each job fee to DAO (swapped to ADA), and **1%** to FeeTank.

***

### 10) Roadmap

| Milestone            | Status / ETA | Deliverables                                                              |
| -------------------- | ------------ | ------------------------------------------------------------------------- |
| **Preview testnet**  | **Done**     | PoI receipts; on‑chain anchors; $AGENT‑only tx (Babel‑fee); escrow events |
| **Mainnet v1**       | **Q4 ’25**   | Epoch roll‑up payouts; slashing; $AGENT‑as‑gas live                       |
| Strategy Marketplace | Q1 ’26       | Mintable quant strategies as NFTs; plug‑and‑play routing of fee flows     |

***

### 11) Risk & mitigation

1. **FeeTank drain / Babel LP risk** → Daily caps + automatic 1% top‑up; emergency ADA fallback if required.
2. **Oracle/Verifier collusion** → Multi‑sig verifiers; open‑source client to recompute hashes.
3. **Liquidity crunch** → Diversify across major Cardano DEXs/aggregators; oracle‑guided quoting; moving‑average price guards and rate‑limiters.
4. **Regulatory clarity** → **$AGENT** used for compute, bonding, and governance; no passive ROI promises.

***

### 12) FAQ

**Q1 — Do I need ADA in my wallet?**\
No. Jobs and fees are **$AGENT‑only**. **Babel‑fees** and the **FeeTank** supply ADA where the chain requires it.

**Q2 — What hardware qualifies to run a pool (EXAMPLES)?**

* **Edge** — Pi 5 / RK3588 / Jetson‑Orin‑Nano (**bond: 5,000 $AGENT**)
* **GPU** — RTX 4090 (≥24GB VRAM) (**bond: 80,000 $AGENT**)
* **Datacenter** — A100/H100 (**bond: 800,000 $AGENT**)\


**Q3 — How are rewards delivered?**\
**Aggregated per epoch** (\~5 days). **StreamDistributor** writes **one** $AGENT payout UTxO per staker. No manual claiming.

**Q4 — Can my bond be slashed?**\
Yes. Incorrect results, missed deadlines, or failed verification can trigger **on‑chain slashing**.

**Q5 — When can I withdraw rewards or un‑bond?**

* **Reward vesting:** Rewards accrue immediately but are **locked for 6 months**; after day 180, withdraw anytime.
* **Un‑bond cool‑down:** **2 epochs** (\~10 days) before bonded $AGENT returns, to prevent “bond‑and‑run.”

**Q6 — Where can I get $AGENT?**\
On **major Cardano DEXs/aggregators**; see market pages such as **TapTools** for venues and pairs.

**Q7 — What if the FeeTank runs out of ADA?**\
Dispatcher can temporarily fall back to normal ADA settlement. The **1% top‑up** keeps it solvent in normal conditions.

**Q8 — How are fees quoted?**\
Quotes are **in $AGENT** based on **model × tokens/time × tier**. The dispatcher shows the quote **before you sign**. No hidden spreads; DAO and FeeTank slices are visible.

**Q9 — How do I run an Inference Pool?** _(≈5‑minute checklist)_

1. **Hardware** — Bring a qualifying device (Edge / GPU / DC).
2. **Wallet** — Fund with the **bond in $AGENT** (no ADA required in your wallet).
3. **Docker** — Install ≥24.0; pull the pool image:

```bash
docker run -d --restart unless-stopped \
  -e CARDANO_NET=mainnet \
  -e WALLET_MNEMONIC="..." \
  ghcr.io/fluxpoint/agent-pool:latest
```

4. **Bond** — Container emits a `bond.cmd`; run it to post the Bond UTxO (Babel‑fee covered).
5. **Networking** — Open **port 3000** (Edge NAT punches via WebSocket tunnel).
6. **Dashboard** — `localhost:8080` shows **Bond ✓ | Jobs | Rewards**; jobs flow from the next epoch.

**Q10 — Can I co‑run Iagon Storage/Compute with my pool?**\
Yes, if you still meet both SLAs.

* **Edge:** not recommended.
* **Multi‑GPU:** pin Iagon to `--gpus="device=1"` (or MIG); keep primary GPU free for inference.
* **A100/H100:** isolate via CUDA context or MIG; NVIDIA MPS will arbitrate.\
  If benchmarks dip, your reputation and rewards will fall; chronic misses can be **slashed**.

***

### 13) Demo: verify the preview run yourself (≈90 seconds)

> Prereqs: Blockfrost _preview_ API key in `BLOCKFROST` and the local receipt file.

**1) Fetch anchor metadata**

```bash
export BLOCKFROST=<your_preview_project_id>
curl -s -H "project_id: $BLOCKFROST" \
https://cardano-preview.blockfrost.io/api/v0/txs/1a697155ec50babafde25d5908c19a6b0c80c068476ef6ff84d0048e3fdef344/metadata | jq .
# label 2222: { action:"anchor", jobId:"petals_1758832859921", resultHash, receiptHash }
```

**2) Check the rolling hash from your receipt**

```bash
jq -r '.transcript.rolling_hash' receipts/petals_1758832859921.poi.receipt.json
# 631166715f80b93133221ca4e62df6c03fdcca693249485676929452b5bf82aa
```

**3) Recompute the signature‑stripped receiptHash**

```bash
node -e "const fs=require('fs');const c=JSON.parse(fs.readFileSync('receipts/petals_1758832859921.poi.receipt.json','utf8'));c.signature='';const h=require('crypto').createHash('sha256').update(JSON.stringify(c)).digest('hex');console.log(h)"
# 14fbdfe780ce6223f549ea514db3493a5b3ea4ceb0cb0f17a09880bd6e33b3e2
```

**4) (Optional) Inspect escrow initiation**

```bash
curl -s -H "project_id: $BLOCKFROST" \
https://cardano-preview.blockfrost.io/api/v0/txs/3a7b52a2852b05fba71301716c11dd486de295bd68d33013d28a3b1069bbe325/metadata | jq .
# label 2222: { action:"initiate", jobId:"...", model:"bigscience/bloom-560m", ... }
```

***

### 14) Compliance

This document is informational and does not constitute an offer to sell tokens. Forward‑looking statements are subject to change at the discretion of Flux Point Studios, Inc.

***

### 15) Contact

* Web: **fluxpointstudios.com**
* X/Twitter: **@fluxpointstudio**
* Discord: **discord.gg/MfYUMnfrJM**

***

**From black boxes to balance sheets:** AI usage becomes an **auditable line item**—with cryptographic receipts and **$AGENT‑denominated** cash flows.
