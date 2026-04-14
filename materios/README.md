---
description: Materios Partner Chain — Overview
---

# Materios

Materios is a **Substrate-based partner chain** purpose-built for certified data anchoring and AI verification. It extends the Proof-of-Inference protocol with a dedicated execution layer optimized for high-throughput receipt storage, blob attestation, and multi-party threshold certification.

## Why a Partner Chain?

Cardano L1 provides strong finality guarantees, but anchoring every individual AI receipt on-chain is cost-prohibitive at scale. Materios solves this by:

- **Batching**: Thousands of PoI receipts are collected, Merkle-hashed, and the root is periodically anchored to Cardano L1.
- **Dedicated throughput**: 6-second block times with lightweight blocks optimized for receipt and blob data.
- **Threshold attestation**: A committee of independent attesters must reach quorum before any batch is considered certified.
- **Dual-token economics**: MATRA (transferable, stakeable) and MOTRA (non-transferable capacity token with generation and decay) provide a fee model that doesn't require users to hold volatile assets.

## Architecture

```
Clients (SDK / API)
    |
    v
Blob Gateway  -->  Cert Daemon Committee (threshold attestation)
    |                       |
    v                       v
Materios Chain          Cardano L1
(Substrate)           (periodic anchors)
```

### Core Components

| Component | Role |
|-----------|------|
| **Materios Node** | Substrate validator/full node. Aura block production + Grandpa BFT finality. |
| **Cert Daemon** | Attestation agent. Signs receipts, participates in threshold certification. |
| **Blob Gateway** | HTTP API for submitting and retrieving certified data blobs. |
| **Anchor Worker** | Bridges Materios batch roots to Cardano L1 metadata transactions. |
| **Explorer** | Web dashboard showing blocks, attestations, committee health, and chain status. |

### Consensus

- **Block authoring**: Aura (authority round-robin, sr25519 keys)
- **Finality**: Grandpa (BFT, ed25519 keys). Tolerates up to `f` faults where `f = (n-1)/3` for `n` authorities.
- **Session keys**: Combined Aura + Grandpa keypair per validator.

### Token Model

| Token | Type | Purpose |
|-------|------|---------|
| **MATRA** | Transferable (12 decimals) | Staking, governance, transfers |
| **MOTRA** | Non-transferable | Capacity/fee token. Generated proportionally to MATRA balance, decays per block. Used to pay transaction fees without spending MATRA. |

### On-Chain Pallets

| Pallet | Function |
|--------|----------|
| `OrinqReceipts` | PoI receipt storage, status tracking, content-based queries |
| `Motra` | MOTRA generation, decay, fee estimation, congestion pricing |
| `Balances` | MATRA transfers and account management |
| `Grandpa` | BFT finality (standard Substrate) |
| `Aura` | Block authoring schedule (standard Substrate) |
| `Sudo` | Governance (2-of-3 multisig, transitioning to Cardano governance contracts) |
| `Multisig` | Multi-signature call dispatch for governance actions |
| `Utility` | Batch call execution through multisig |

## Current Network

- **Chain**: `materios_staging`
- **Runtime version**: 115
- **Validators**: 5 authorities (+ external validators joining)
- **Attestation committee**: 10 members, threshold 2-of-10
- **Governance**: 2-of-3 multisig sudo
- **Validator keys**: Production keypairs (rotated from development accounts, April 12 2026)
- **Cardano anchoring**: Mainnet, label `8746`
- **Explorer**: [materios.fluxpointstudios.com/explorer](https://materios.fluxpointstudios.com/explorer/)

## Next Steps

- [Mainnet Roadmap](mainnet-roadmap.md) — Current status and remaining milestones for mainnet
- [Cardano L1 Anchoring](cardano-anchoring.md) — Metadata format for Materios checkpoint transactions on Cardano mainnet
- [Operator Guide](operator-guide.md) — Become an attestation operator with a single command
- [Node Requirements](node-requirements.md) — Hardware, software, and configuration for running a full Materios node
