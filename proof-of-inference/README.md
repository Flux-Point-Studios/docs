# Orynq — Verifiable AI

**Orynq** is Flux Point Studios' verifiable-AI stack. It produces cryptographic proof that AI systems did what they claimed — signed receipts for AI behavior, threshold-certified by the [Materios blockchain](../materios/README.md) and immutably anchored to Cardano L1.

> **Orynq supersedes Proof-of-Inference (PoI).** The earlier $AGENT-denominated PoI protocol — job fees in $AGENT, staker fee-shares, inference-pool bonds, and PoI governance — is retired and will not ship. $AGENT itself is being consolidated into cMATRA under the [cMATRA Token Merger](../materios/cmatra-token-merger/README.md). On Materios, Orynq services are paid as ordinary network fees (MATRA / MOTRA capacity), and network security is compensated with validator and attestor block rewards — see the [PoI Litepaper (Legacy)](litepaper.md) page for the full mapping.

## Overview

Orynq addresses the fundamental challenge of AI transparency: how do you prove what an AI did, with which inputs, producing which outputs? The stack combines:

- **Cryptographic Receipts**: Every observation or process trace generates a signed receipt with deterministic, content-addressed hashes
- **Threshold Certification**: A committee of independent attesters reaches quorum on Materios before a receipt batch is certified
- **On-Chain Anchoring**: Compact fingerprints are immutably anchored to Cardano (metadata label 8746)
- **Independent Verification**: Anyone can verify the full chain of custody from receipt fields back to a Cardano transaction, without trusting any single party

## Components

<table data-view="cards">
<thead><tr><th></th><th></th></tr></thead>
<tbody>
<tr>
<td><strong>Orynq SDK</strong></td>
<td>Developer toolkit for anchoring AI process traces via Materios to Cardano</td>
</tr>
<tr>
<td><strong>Orynq Observe</strong></td>
<td>SDK for publishing cryptographically-attested AI model observation receipts to a public registry</td>
</tr>
<tr>
<td><strong>Anchor-as-a-Service</strong></td>
<td>Managed API for anchoring AI process traces without running infrastructure</td>
</tr>
</tbody>
</table>

## Use Cases

### Enterprise AI Compliance
Create auditable records of AI decision-making for regulatory compliance and internal governance.

### AI Agent Verification
Prove that autonomous AI agents executed their tasks correctly and within defined parameters.

### Model Inference Auditing
Track which models processed which data, enabling accountability in AI pipelines.

### Research Reproducibility
Anchor experimental AI runs to enable independent verification of research results.

## Quick Links

- [Orynq SDK](orynq-sdk.md) - Developer documentation
- [Orynq Observe](../orynq/observe.md) - Attested model observation receipts
- [Anchor-as-a-Service](https://fluxpointstudios.com/anchor) - Try the live demo
- [PoI Litepaper (Legacy)](litepaper.md) - The superseded PoI-era specification, kept for the record
- [GitHub](https://github.com/Flux-Point-Studios/orynq-sdk) - Source code
