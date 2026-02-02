# Proof-of-Inference

Proof-of-Inference (PoI) is Flux Point Studios' framework for **verifiable AI** on blockchain. It provides cryptographic proof that AI systems did what they claimed—with immutable audit trails stored on Cardano.

**Orynq** is the umbrella platform that powers Proof-of-Inference anchoring and other AI verification services.

## Overview

PoI addresses the fundamental challenge of AI transparency: how do you prove what an AI did, with which inputs, producing which outputs? Our solution combines:

- **Cryptographic Receipts**: Every AI inference generates a signed receipt with deterministic hashes
- **On-Chain Anchoring**: Compact fingerprints are immutably stored on Cardano (label 2222)
- **Independent Verification**: Anyone can verify proofs using only the transaction hash
- **Economic Security**: Operators stake $AGENT tokens; misbehavior triggers slashing

## Components

<table data-view="cards">
<thead><tr><th></th><th></th></tr></thead>
<tbody>
<tr>
<td><strong>Litepaper</strong></td>
<td>Technical specification of the PoI protocol, economics, and architecture</td>
</tr>
<tr>
<td><strong>Orynq SDK</strong></td>
<td>Developer toolkit for integrating PoI anchoring into your applications</td>
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

- [Litepaper](litepaper.md) - Full technical specification
- [Orynq SDK](poi-sdk.md) - Developer documentation
- [Anchor-as-a-Service](https://fluxpointstudios.com/anchor) - Try the live demo
- [GitHub](https://github.com/Flux-Point-Studios/orynq-sdk) - Source code
