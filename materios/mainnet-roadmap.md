---
description: Materios mainnet readiness tracker — current status and remaining milestones
---

# Mainnet Roadmap

Materios is currently running as a **preprod network** (`materios_preprod`, runtime v117) with live data, real game integrations, and mainnet Cardano anchoring. This page tracks what's done and what remains before the network is designated as mainnet.

***

## Current Status: Preprod Network

The Materios preprod network (`materios_preprod`) is fully operational with real workloads:

- **Live game integration** — Clay Monster Dash submits certified receipts through the full pipeline
- **Cardano mainnet anchoring** — Checkpoint transactions are submitted to Cardano L1 with label `8746` ([view format](cardano-anchoring.md))
- **Permissionless attestation** — External operators can join the committee with a single command ([operator guide](operator-guide.md))
- **~12-second certification** — Receipts go from submitted to certified in approximately 12 seconds
- **Clean genesis** — Runtime v117 with all fixes baked in, no WASM overrides needed
- **GRANDPA finality** — Working with 3 validators (Gemtek + 2 GMKtec Ultra 6 mini PCs)

The preprod network uses the same codebase, pallets, and protocols that will run on mainnet. The transition is about token economics and governance, not a rewrite. The previous staging/preview network is deprecated.

***

## Completed Milestones

### Infrastructure & Pipeline

| Milestone | Status | Details |
|-----------|--------|---------|
| Substrate chain (Aura + GRANDPA) | Done | 3 validators (preprod), 6-second blocks, BFT finality |
| Receipt pipeline (submit → certify → anchor) | Done | End-to-end proven, ~12s certification |
| Cardano L1 anchoring | Done | Mainnet metadata TXs, label `8746`, v2 format |
| Blob gateway + locator registry | Done | Off-chain data storage with integrity verification |
| Content validation schemas | Done | Per-game plausibility checks (Clay Monster Dash v1 live) |
| Explorer dashboard | Done | [fluxpointstudios.com/materios/explorer](https://fluxpointstudios.com/materios/explorer) (Preprod/Preview toggle) |

### Attestation Committee

| Milestone | Status | Details |
|-----------|--------|---------|
| Threshold attestation | Done | Multi-party certification with configurable quorum |
| Permissionless attestor onboarding | Done | One-command install, auto-join committee, auto-register |
| Health watchdog for operators | Done | Discord/email alerting for daemon issues |
| External attestors onboarding | In progress | Multiple external operators joining the committee |

### Governance

| Milestone | Status | Details |
|-----------|--------|---------|
| Multisig governance | Done | Runtime v114 — `pallet-multisig` + `pallet-utility`. Sudo requires 2-of-3 approval. |
| MOTRA projected balance | Done | Runtime v115 — `motra_getBalance` RPC returns projected balance for fresh accounts immediately. |
| Preprod chain launch | Done | Runtime v117 — Clean genesis (`materios_preprod`), 3 validators, no WASM overrides, GRANDPA finality working. |
| Validator key rotation | Done | Production keypairs (mnemonic-derived, April 12 2026). External validators joining. |
| Cardano governance contracts | Planned | On-chain voting by MATRA/cMATRA holders via Cardano smart contracts |

### Token Economics

| Milestone | Status | Details |
|-----------|--------|---------|
| Dual-token model (MATRA + MOTRA) | Done | MATRA for staking/governance, MOTRA for fees (generation + decay) |
| cMATRA token merger | Planned | 7 legacy Cardano assets → cMATRA ([details](https://docs.fluxpointstudios.com/materios-partner-chain/cmatra-token-merger)) |
| Validator rewards (stake-weighted) | Planned | Transition from block-count to stake-weighted distribution |

***

## Remaining Milestones

### 1. cMATRA Token Merger

Seven legacy Cardano assets (AGENT, SHARDS, and 5 NFT collections) will consolidate into **cMATRA**, a Cardano-native transitional token. cMATRA will bridge to become native **MATRA** on the Materios mainnet.

- Maximum supply: 1 billion cMATRA
- 15% validator reserve (150M)
- 85% public redemption pool (850M)
- 6-month public redemption window

See the full merger specification: [cMATRA Token Merger](https://docs.fluxpointstudios.com/materios-partner-chain/cmatra-token-merger)

### 2. Stake-Weighted Validator Rewards

Transition from the current block-count reward model to stake-weighted distribution, aligning validator incentives with network security contribution.

### 3. Cardano Governance

Long-term governance will be managed through Cardano smart contracts, enabling MATRA/cMATRA holders to participate in on-chain governance proposals. This aligns with the Cardano Partner Chains vision where ADA holders benefit from partner chain participation.

***

## Network Parameters

| Parameter | Current (preprod) | Mainnet target |
|-----------|------------------|----------------|
| Runtime version | 117 | TBD |
| Chain ID | `materios_preprod` | `materios` |
| Block time | 6 seconds | 6 seconds |
| Finality | GRANDPA BFT | GRANDPA BFT |
| Validators | 3 (Gemtek + 2 GMKtec) | 7–15+ |
| Attestation threshold | 2-of-N | 2-of-N (scales with committee) |
| Cardano anchoring | Mainnet, label `8746` | Same |
| Governance | 2-of-3 multisig | Cardano governance contracts |
| WASM overrides | None needed | None |

***

## Timeline

The mainnet transition depends primarily on the cMATRA token merger, which requires a 6-month public redemption window. The merger defines the initial MATRA distribution, which is a prerequisite for stake-weighted rewards and Cardano governance.

All infrastructure milestones (pipeline, anchoring, attestation, multisig governance, validator key rotation) are already complete. The remaining work is token economics and decentralization.

***

## Links

- [Explorer](https://fluxpointstudios.com/materios/explorer) — Live network dashboard (Preprod/Preview toggle)
- [Operator Guide](operator-guide.md) — Run an attestor node
- [Cardano L1 Anchoring](cardano-anchoring.md) — Metadata format for anchor transactions
- [Game Integration](game-integration.md) — Add certified anti-cheat to your game
- [cMATRA Token Merger](https://docs.fluxpointstudios.com/materios-partner-chain/cmatra-token-merger) — Legacy asset consolidation
