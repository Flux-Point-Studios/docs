---
description: >-
  Legacy document. The PoI protocol and its $AGENT economics are superseded by
  Orynq on the Materios blockchain and by the cMATRA Token Merger.
---

# Proof-of-Inference Litepaper (Legacy)

> **Superseded — kept for the record.** Proof-of-Inference (PoI) evolved into **[Orynq](README.md)**, Flux Point Studios' verifiable-AI stack on the [Materios blockchain](../materios/README.md). The $AGENT-based economics described in the original litepaper — job fees paid in $AGENT, the 70/28/1/1 fee split, epoch-rolled staker payouts, inference-pool bonds, and $AGENT-weighted governance — are **retired and will not ship**. There are no $AGENT rewards, staker fee-shares, or PoI governance going forward. $AGENT is being consolidated into **cMATRA** under the [cMATRA Token Merger](../materios/cmatra-token-merger/README.md); holders can surrender it at the [official portal](https://fluxpointstudios.com/matra-merger) until **November 28, 2026**.

***

## What PoI was

PoI (2025) was the original framework for **verifiable AI on Cardano**: deterministic inference runs (fixed seed/params, rolling hashes) emitting signed receipts, with compact fingerprints anchored on-chain so anyone could later verify who ran what, with which model, from which inputs, producing which outputs. A preview-testnet demonstration ran decentralized inference on a Petals-based swarm, generated a signed PoI receipt, and anchored it to Cardano under metadata label 2222.

The verification idea was right. The token economics wrapped around it were not — they tied a verification protocol to a bespoke fee-and-rewards token, which is exactly the kind of app-by-app utility scheme the cMATRA merger retires.

## Where each piece went

| PoI-era design                                        | Current equivalent                                                                                                          |
| ----------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Signed inference receipts, anchored on Cardano (2222) | **Orynq receipts** — threshold-attested on Materios, batch-anchored to Cardano L1 under label **8746**                       |
| $AGENT as gas / Babel-fee job payments                | Ordinary **Materios network fees** — MATRA holdings generate MOTRA capacity, which pays per-receipt fees                     |
| 28% staker fee-share, epoch-rolled payouts            | **Retired.** Network security is compensated with validator and attestor **block rewards** from published emission schedules |
| $AGENT inference-pool bonds and slashing              | **Attestor bond/slash economics** on Materios (Attestor Emissions sub-bucket)                                                |
| PoI governance ($AGENT-weighted voting)               | **Retired.** Materios chain governance, carried by MATRA as governance decentralizes                                         |
| $AGENT token                                          | Consolidating into **cMATRA → MATRA**, the network token of the Materios blockchain, via the merger                          |

## Where to go instead

* **[Orynq overview](README.md)** — the current verifiable-AI stack
* **[Orynq SDK](orynq-sdk.md)** — process-trace anchoring and Anchor-as-a-Service
* **[Orynq Observe](../orynq/observe.md)** — attested AI model observation receipts and the public registry
* **[Materios Partner Chain](../materios/README.md)** — the chain that certifies and anchors Orynq receipts
* **[cMATRA Token Merger](../materios/cmatra-token-merger/README.md)** — how to trade $AGENT for the network token

***

*The full original litepaper text (October 14, 2025) remains available in the [git history](https://github.com/Flux-Point-Studios/docs/commits/main/proof-of-inference/litepaper.md) of this page. This document is informational and does not constitute an offer to sell tokens.*
