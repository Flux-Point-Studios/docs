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
- **Public Redemption Pool:** 722.5M (72.25%)
- **Network Incentives Reserve:** 277.5M (27.75%) — split into five purpose-built sub-buckets:
  - Validator Emissions — 115M (11.5%)
  - Attestor Emissions — 65M (6.5%)
  - Ecosystem Treasury — 40M (4%)
  - Strategic Allocation — 30M (3%) — 12-month cliff + 36-month linear vesting
  - Liquidity — 27.5M (2.75%) — POL, bridge peg reserve, CLOB maker rebates
- 6-month public redemption window

See the full merger specification: [cMATRA Token Merger](https://docs.fluxpointstudios.com/materios-partner-chain/cmatra-token-merger)

### 2. Stake-Weighted Validator Rewards

Transition from the current block-count reward model to stake-weighted distribution, aligning validator incentives with network security contribution. Target scale: **1,000 validators + 3,000 attestors** at steady state, with per-validator target reward of ~5K MATRA/year and per-attestor target of ~2K MATRA/year.

### 3. Cardano Governance

Long-term governance will be managed through Cardano smart contracts, enabling MATRA/cMATRA holders to participate in on-chain governance proposals. This aligns with the Cardano Partner Chains vision where ADA holders benefit from partner chain participation.

### 4. Launch Liquidity

Seed the cMATRA/ADA and cMATRA/USDM markets on **SaturnSwap CLOB** (primary) with secondary AMM listings on Minswap and WingRiders at launch. Funded by the 27.5M Liquidity sub-bucket plus a strategic pair-sourcing arrangement:

- **SaturnSwap CLOB:** $250K book depth within 2% of mid at launch; resting orders at 25/50/100/200/400 bps from mid.
- **Minswap AMM:** $75–100K depth.
- **WingRiders AMM:** $25–50K depth.
- **Maker-rebate program:** 5M MATRA over 24 months, 50% annual decay, ~15 bps rebate on 30 bps maker fee, 20% cap per MM address with two-sided-quote requirements.

### 5. Strategic Fundraise

A **Strategic Allocation** of 30M MATRA (3% of supply) is reserved for institutional partners who commit capital ahead of public launch. Allocation terms include a 12-month cliff followed by 36-month linear vesting, with no pre-cliff liquidity. Proceeds are intended to fund Protocol-Owned Liquidity seeding, security audits, and post-window team runway. Partner identities and commitment amounts will be disclosed on-chain at mint time.

***

## Network Parameters

| Parameter | Current (preprod) | Mainnet target |
|-----------|------------------|----------------|
| Runtime version | v3 (IOG partner-chains / Minotaur) | TBD |
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

### Long-term sustainability

The Network Incentives Reserve is sized as a **first-year runway** rather than a permanent ceiling. Once the network is live under fee load, **fee recycling** kicks in: every transaction's fee is split 40% to a block-author pot, 30% to the Attestor Emissions pot, 20% to the Ecosystem Treasury, and 10% burned. This means validator and attestor income has two compounding sources — the initial emissions schedule plus ongoing fee flow — and the reserves extend proportionally with network usage.

At the 1,000-validator + 3,000-attestor target scale, the combined annual reward requirement is approximately 11M MATRA/year. The 180M combined Validator + Attestor Emissions sub-buckets provide ~16 years of runway at 0% fee coverage, ~58 years at 75% fee coverage, and become perpetual at 100% fee coverage. With the unredeemed cMATRA rollover pallet (6-month post-launch trigger), residual redemption-pool balances flow back into the reserve sub-buckets on a governance-directed split, making the reserve effectively perpetual at any realistic level of network activity.

***

## Links

- [Explorer](https://fluxpointstudios.com/materios/explorer) — Live network dashboard (Preprod/Preview toggle)
- [Operator Guide](operator-guide.md) — Run an attestor node
- [Cardano L1 Anchoring](cardano-anchoring.md) — Metadata format for anchor transactions
- [Game Integration](game-integration.md) — Add certified anti-cheat to your game
- [cMATRA Token Merger](https://docs.fluxpointstudios.com/materios-partner-chain/cmatra-token-merger) — Legacy asset consolidation
