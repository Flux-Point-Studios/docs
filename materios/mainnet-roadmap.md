---
description: Materios mainnet readiness tracker — current status and remaining milestones
---

# Mainnet Roadmap

Materios is currently running as a **preprod network** (`materios_preprod_v6`, runtime spec 235) with live data, real game integrations, and mainnet Cardano anchoring. This page tracks what's done and what remains before the network is designated as mainnet.

***

## Current Status: Preprod Network

The Materios preprod network (`materios_preprod_v6`) is fully operational with real workloads:

- **Live game integration** — Clay Monster Dash submits certified receipts through the full pipeline
- **Cardano mainnet anchoring** — Checkpoint transactions are submitted to Cardano L1 with label `8746` ([view format](cardano-anchoring.md))
- **Permissionless attestation** — External operators join the attestation committee with a single command ([operator guide](operator-guide.md))
- **Permissionless SPO validators** — Cardano preprod SPOs can register via `smart-contracts register` and compete for the open registered committee seat (D = `(15, 1)`); see [SPO Onboarding](spo-onboarding.md)
- **~12-second certification** — Receipts go from submitted to certified in approximately 12 seconds
- **Runtime overrides** — IOG IDP-None fallback + Ariadne output dedup ship as `--wasm-runtime-overrides`; upstream fixes pending
- **GRANDPA finality** — Working with 4 permissioned validators (Gemtek, 2× GMKtec Ultra 6, MacBook Pro M1 native arm64)

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
| Preprod chain launch | Done | Runtime spec 235 — clean genesis (`materios_preprod_v6`), 5-seat committee, GRANDPA finality working. Operators bootstrap from the current-room snapshot. |
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
  - Strategic Allocation — 30M (3%) — long-term on-chain vesting
  - Liquidity — 27.5M (2.75%) — POL, bridge peg reserve, CLOB maker rebates
- 6-month public redemption window (open May 28 – November 28, 2026)

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

A **Strategic Allocation** of 30M MATRA (3% of supply) is reserved for institutional partners who commit capital ahead of public launch. Allocations are subject to long-term on-chain vesting, with no liquidity ahead of schedule. Proceeds are intended to fund Protocol-Owned Liquidity seeding, security audits, and post-window team runway. Partner identities and commitment amounts will be disclosed on-chain at mint time.

***

## Network Parameters

| Parameter | Current (preprod) | Mainnet target |
|-----------|------------------|----------------|
| Runtime version | spec 235, tx version 4 (MATRA 6-dec, MOTRA 15-dec) | TBD |
| Chain ID | `materios_preprod_v6` | `materios` |
| Block time | 6 seconds | 6 seconds |
| Finality | GRANDPA BFT | GRANDPA BFT |
| Validators | 4 permissioned cores + 1 SPO-registered (D = `(15, 1)`) | 7–15+ |
| Attestation threshold | 2-of-N | 2-of-N (scales with committee) |
| Cardano anchoring | Mainnet, label `8746` | Same |
| Governance | 2-of-3 multisig | Cardano governance contracts |
| WASM overrides | None needed | None |

***

## Timeline

The mainnet transition depends primarily on the cMATRA token merger, which requires a 6-month public redemption window (closes November 28, 2026). The merger defines the initial MATRA distribution, which is a prerequisite for stake-weighted rewards and Cardano governance.

All infrastructure milestones (pipeline, anchoring, attestation, multisig governance, validator key rotation) are already complete. The remaining work is token economics and decentralization.

### Long-term sustainability

Validator and attestor rewards are paid from the published **Network Incentives Reserve emission schedules** (115M Validator Emissions, 65M Attestor Emissions); the attestor pot (`mat/attr`) is the designated accrual account for attestor-side economics. Transaction fees themselves are paid in **MOTRA** — generated from MATRA holdings, decaying if unused, consumed when spent — so fee flow never burns, mints, or dilutes MATRA. Live pot balances, the v5.1 allocation table, and active vesting schedules are visible on the [Materios explorer's Tokenomics State panel](https://fluxpointstudios.com/materios/explorer#overview).

At the 1,000-validator + 3,000-attestor target scale, the combined annual reward requirement is approximately 11M MATRA/year. The 180M combined Validator + Attestor Emissions sub-buckets provide ~16 years of runway at that scale on emissions alone — MOTRA fees are burned when spent and never fund rewards. With the unredeemed cMATRA rollover pallet (6-month post-launch trigger), residual redemption-pool balances flow back into the reserve sub-buckets on a governance-directed split, making the reserve effectively perpetual at any realistic level of network activity.

***

## Links

- [Explorer](https://fluxpointstudios.com/materios/explorer) — Live network dashboard (Preprod/Preview toggle)
- [Operator Guide](operator-guide.md) — Run an attestor node
- [Cardano L1 Anchoring](cardano-anchoring.md) — Metadata format for anchor transactions
- [Game Integration](game-integration.md) — Add certified anti-cheat to your game
- [cMATRA Token Merger](https://docs.fluxpointstudios.com/materios-partner-chain/cmatra-token-merger) — Legacy asset consolidation
