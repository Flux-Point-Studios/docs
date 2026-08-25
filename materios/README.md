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
| **MATRA** | Transferable (6 decimals) | Staking, governance, transfers |
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

### Preprod (active)

> **🧪 Preprod is a public testnet, not mainnet.** Native token on this network is **tMATRA** — testnet tokens with no economic value. Operating a node here is for testing, demonstrating uptime, and previewing mainnet workflows; it is not a revenue stream. Mainnet (with real MATRA + economic rewards) launches per the [Mainnet Roadmap](mainnet-roadmap.md).

- **Chain**: `materios_preprod_v6` (`system_chain` reports `Materios Preprod v6`)
- **Runtime version**: spec **235**, transaction version **4**
- **Token decimals**: MATRA 6, MOTRA 15
- **Validators**: 5-seat committee — 4 FPS permissioned cores + 1 SPO-registered seat
- **D-parameter**: `(15, 1)` — 15 permissioned seats, 1 registered seat
- **Block time**: 6 seconds
- **Finality**: GRANDPA (working)
- **Governance**: 2-of-3 multisig sudo (transferred from //Alice)
- **Cardano anchoring**: Mainnet, label `8746`
- **RPC**: `wss://materios.fluxpointstudios.com/preprod-rpc`
- **Gateway**: `https://materios.fluxpointstudios.com/preprod-blobs`
- **Explorer**: [fluxpointstudios.com/materios/explorer](https://fluxpointstudios.com/materios/explorer)
- **Genesis hash**: `0x0e46e33f639a56cc8780fd871d9a15e16d99af248526f907cb560cb40849f7bf`
- **Node bootstrap**: restore the current-room snapshot, not a from-genesis replay — see [Current-snapshot bootstrap](current-snapshot-bootstrap.md).

> **Integrating a wallet or SDK?** Materios has two deviations from a stock Substrate
> chain that will break a naive integration: a non-standard fee extension and an SS58
> prefix mismatch. Both are documented in [Wallet & SDK Integration](wallet-integration.md).
> Read that page before writing signing code.
- **Attestor install (permissionless)**: `curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install.sh | bash -s -- --mode attestor`
- **SPO Validator install**: see [SPO Onboarding](spo-onboarding.md).

### Preview (deprecated)

The original preview/staging network (`materios_staging`, runtime 115, 5 validators) is deprecated. All new operators should join the **preprod** chain. Preview will be sunset once all integrations have migrated.

## Next Steps

- [Mainnet Roadmap](mainnet-roadmap.md) — Current status and remaining milestones for mainnet
- [Cardano L1 Anchoring](cardano-anchoring.md) — Metadata format for Materios checkpoint transactions on Cardano mainnet
- [Operator Guide](operator-guide.md) — Become an attestation operator with a single command
- [Node Requirements](node-requirements.md) — Hardware, software, and configuration for running a full Materios node
