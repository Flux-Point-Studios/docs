---
description: Overview of how to run a Materios partner-chain validator. Lead with the trustless SPO path; permissioned and attestor paths covered briefly with links.
---

# Operator Guide

> **Preprod is a public testnet.** Rewards on this network are paid in **tMATRA** (test tokens, no economic value). See [Mainnet Roadmap](mainnet-roadmap.md) for the path to real-MATRA rewards.

A Materios validator is a Substrate node that produces blocks and signs finality for the Materios partner-chain, with committee membership rotated each Cardano `mc_epoch` (~5 days on preprod) by IOG's Ariadne selection algorithm. Settlement anchors back to Cardano L1; rewards accrue on Materios.

## Paths to validate

| Path | Who it's for | Onboarding | Time to first block |
|---|---|---|---|
| **Trustless SPO** *(canonical)* | Anyone with a Cardano preprod stake pool | Register a candidate tx on Cardano L1 binding your pool to a sidechain key. No FPS approval. | One `mc_epoch` boundary after your registration's snapshot becomes `set` — ~10 days on preprod. |
| Permissioned | Vetted operators on the FPS allowlist | Send three pubkeys to FPS; we submit `upsert-permissioned-candidates`. | ~3 hours after the Cardano tx lands. |
| Attestor | Anyone | One-line installer, runs `cert-daemon` only — does not run a Materios node. | Immediate. |

**The trustless SPO path is the right answer for new operators.** Skip the FPS approval loop, own your registration end-to-end, and earn the same rewards as a permissioned validator once selected. The permissioned path exists for the initial backbone and stays available for vetted partners; new operators should not request it.

If you don't have a Cardano stake pool, you have two options: register one (the [SPO Onboarding](spo-onboarding.md) recipe is end-to-end including the pool itself), or run an attestor instead (no Cardano-side prerequisite, attestation rewards only).

## Hardware

Minimum for a self-hosted stack (cardano-node + db-sync + Postgres + Ogmios + materios-node on one host):

- **4 vCPU**
- **8 GB RAM**
- **NVMe SSD, ≥ 100 GB** (db-sync alone grows past 40 GB on preprod)
- **Inbound TCP 30333** reachable from the public internet (bootnode dial-back + peer gossip)
- **glibc ≥ 2.38**, systemd, x86_64 Linux (Ubuntu 22.04/24.04 LTS or Debian 12)

Hetzner CX52 (~€21/mo) is the floor. Do not attempt on a 4-GB host — Postgres + db-sync + cardano-node will OOM-kill each other under preprod load.

Hosted db-sync (TxPipe Dolos, Demeter.run, Blockfrost) cuts the requirement to **4 vCPU / 8 GB / 40 GB** because cardano-node and Postgres move off-host.

**Not supported:** WSL2 (NAT breaks libp2p), Docker Desktop amd64-on-arm64 (Rosetta breaks libp2p noise handshake), Alpine / musl-libc (glibc-linked binary).

## Timeline

| Day | Action |
|---|---|
| **0** | Register Cardano preprod stake pool (skip if already registered). |
| **0** | Generate Materios sidechain/aura/grandpa keys. |
| **0** | Submit partner-chain candidate registration tx on Cardano L1. |
| **+2 epochs (~10 days)** | Stake snapshot becomes `set`; Ariadne starts considering your candidate. |
| **Next `mc_epoch` boundary** | First selection cycle. If drawn, your node enters `currentCommittee`. |
| **+~6s per slot** | You produce your first block. |

Probability of being drawn each epoch is your active delegated stake divided by the total stake of all registered candidates, weighted by the D-parameter's registered-bucket count. The current preprod D-parameter is `(5, 2)` — 5 permissioned + 2 registered seats.

## Next step

For the concrete step-by-step recipe, see [SPO Onboarding](spo-onboarding.md).

For chain spec, ports, and per-component memory budgets, see [Node Requirements](node-requirements.md).

## Attestor (permissionless)

Anyone can run an attestor; no Cardano-side prerequisite, no Materios validator node, no approval step. The cert-daemon polls Materios for new receipts, verifies blob integrity, signs an attestation, and submits on-chain. Each attestation that helps a receipt cross the threshold pays 10 tMATRA.

```bash
curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install.sh \
  | bash -s -- --mode attestor --label my-attestor-01
```

Requirements: Docker 20+, 1 vCPU, 512 MB RAM, 1 GB disk, outbound HTTPS+WSS. macOS / Windows + Docker Desktop both work for the attestor role (the image is multi-arch).

Full details, double-clickable installers, and operating commands: [materios-operator-kit](https://github.com/Flux-Point-Studios/materios-operator-kit).

## Permissioned validator (legacy / FPS-vetted)

If you have a standing arrangement with Flux Point Studios — partner SPO, ecosystem grantee, or invited operator — the permissioned path is faster: send your three pubkeys, we submit a Cardano-side `upsert-permissioned-candidates` tx, and your node is eligible ~3 hours later.

Generate keys with `partner-chains-node wizards generate-keys`, DM the three pubkeys to FPS, then run the bootstrap script:

```bash
curl -fsSL https://materios.fluxpointstudios.com/releases/bootstrap-validator.sh -o bootstrap-validator.sh
curl -fsSL https://materios.fluxpointstudios.com/releases/SHA256SUMS | grep bootstrap-validator.sh
sha256sum bootstrap-validator.sh   # must match
chmod +x bootstrap-validator.sh

sudo -E ./bootstrap-validator.sh \
  --operator-label <your-label> \
  --db-sync 'postgres://<user>:<pw>@127.0.0.1:5432/cexplorer' \
  --ogmios ws://127.0.0.1:1337 \
  --aura-pubkey 0x<your-aura-pub>
```

The script downloads the v6 binary, seeds the keystore, restores the latest snapshot from `https://materios.fluxpointstudios.com/operator-snapshots/preprod/latest.json`, writes a systemd unit, and starts the validator.

New operators: **prefer the trustless SPO path.** It puts you in control of your own onboarding without an FPS bottleneck.

## Rewards

| Pool | Reserve (mainnet design) | Per-event reward | Earners |
|---|---|---|---|
| Block production | 150M MATRA | Per-block credit, distributed at era end (~24 h) | Validators (SPO + permissioned) |
| Attestation | 50M MATRA | 10 per certification per signer, instant | All signers (validators + attestors) |

On preprod the same pools mint tMATRA at the same rates. tMATRA is not exchangeable; preprod participation is for operational hardening and leaderboard credit ahead of mainnet.

## Security

- Mnemonics are generated locally and never leave your host.
- Session keys live only in your node's local keystore.
- Bind node RPC to 127.0.0.1; do not use `--rpc-external` on a public IP.
- Cardano cold keys never leave your SPO host; FPS tooling never asks for them.

## Getting help

- Chain / runtime issues: [Flux-Point-Studios/materios](https://github.com/Flux-Point-Studios/materios/issues)
- Operator-kit / attestor: [Flux-Point-Studios/materios-operator-kit](https://github.com/Flux-Point-Studios/materios-operator-kit/issues)
- Cardano-side registration: [preprod.cexplorer.io](https://preprod.cexplorer.io/)
- Live committee + leaderboard: [explorer](https://fluxpointstudios.com/materios/explorer)
