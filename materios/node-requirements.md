---
description: Hardware, software, keys, and infrastructure needed to run a Materios validator (SPO or permissioned), attestor, or full node on the preprod network.
---

# Node Requirements

> **🧪 Preprod is a public testnet, not mainnet.** Rewards on this network are paid in **tMATRA** — testnet tokens with no economic value. Running a preprod node is for testing the operator workflow and demonstrating uptime; it is not a revenue stream. Mainnet (with real MATRA + economic rewards) has not launched yet — see [Mainnet Roadmap](mainnet-roadmap.md).

Materios is a Cardano Partner Chain. Committee selection uses Ariadne with two tracks: a **permissioned-candidate allowlist** managed via partner-chains governance, and an **SPO-registered pool** weighted by preprod-ADA delegation. The current preprod D-parameter is `(8, 0)` — 8 permissioned seats, 0 SPO seats — until enough external SPOs register and the parameter is bumped. **You can run a validator without being a Cardano SPO** by joining the permissioned-candidate list (see below).

There are four roles:

| Role | What it does | Requires Cardano SPO? |
|------|---------|---|
| **SPO Validator** | Produces blocks, votes on finality, signs attestations. Earns **tMATRA** (testnet, no economic value) on preprod; real MATRA after mainnet launch. Selected by Ariadne weighted by preprod-ADA stake. | **Yes** — register on Cardano preprod via `partner-chains-node smart-contracts register`. |
| **Permissioned Validator** | Same chain duties as SPO Validator. Same **tMATRA** preprod reward; same future mainnet earning curve. Selected from the operator-managed permissioned-candidate list. | **No** — submit your sidechain/Aura/Grandpa pubkeys to Flux Point Studios; we add you to the permissioned-candidates list via governance tx. No Cardano stake pool, no tADA, no 10-day stake-snapshot wait. |
| **Attestor** | Verifies blobs and signs attestations. Earns **10 tMATRA** (testnet, no economic value) per certified receipt on preprod. | No. Permissionless, one-liner install. |
| **Full Node** | RPC + archive / read-only peer. | No. |

See the [Operator Guide](operator-guide.md) for end-to-end setup flows.

## Current Network

| Field | Value |
|---|---|
| Chain | `Materios Preprod v6` |
| Chain ID | `materios_preprod_v6` |
| Genesis hash | `0x0e46e33f639a56cc8780fd871d9a15e16d99af248526f907cb560cb40849f7bf` |
| Cardano genesis UTXO | `13313ea0119e0c4330f64f1809159064a371a1bbf2050b1fe13d5492280dca50#0` |
| spec_version | 211+ (live runtime upgrades; check `state_getRuntimeVersion`) |
| Block time | 6 seconds |
| Session length | 60 blocks (~6 minutes) |
| Cardano L1 | Preprod testnet |
| Main-chain epoch length | 5 days |
| D-parameter | `(8, 0)` — 8 permissioned seats, 0 SPO seats (preprod). Will move to a mixed (P, R) split once external SPOs are registered. |
| Public RPC (WSS) | `wss://materios.fluxpointstudios.com/preprod-rpc` |
| Explorer | [fluxpointstudios.com/materios/explorer](https://fluxpointstudios.com/materios/explorer) |

> **🔁 v6 chain reset, 2026-04-28.** Preprod was reset from v5 to v6 to fix accumulated state issues. v5 registrations did NOT carry over — operators previously on v5 must re-onboard. The v5 → v6 incident notes are documented in our internal runbook.

The **8 permissioned seats** are filled from the partner-chains permissioned-candidate list. **External operators can apply for a permissioned seat** without becoming a Cardano SPO — see [Operator Guide → Permissioned Validator](operator-guide.md#permissioned-validator-non-spo). SPO-registered seats are not enabled on preprod yet; they'll be opened once a quorum of external SPOs are registered.

## Static Asset Distribution

Everything an operator needs to download lives under `materios.fluxpointstudios.com`:

| Asset | URL | sha256 |
|---|---|---|
| Bootstrap installer (canonical) | [`releases/bootstrap-validator.sh`](https://materios.fluxpointstudios.com/releases/bootstrap-validator.sh) | see SHA256SUMS |
| Node binary v6 (x86_64 Linux) | [`releases/materios-node-v6-x86_64-linux`](https://materios.fluxpointstudios.com/releases/materios-node-v6-x86_64-linux) | `dda4f3a7…58ebc` |
| Chain spec v6 (raw JSON) | [`releases/chain-spec-v6-raw.json`](https://materios.fluxpointstudios.com/releases/chain-spec-v6-raw.json) | `77877aa6…65aaa` |
| Data snapshot (current rocksdb pointer) | [`operator-snapshots/preprod/latest.json`](https://materios.fluxpointstudios.com/operator-snapshots/preprod/latest.json) | sha256 inside the manifest |
| SHA256SUMS manifest | [`releases/SHA256SUMS`](https://materios.fluxpointstudios.com/releases/SHA256SUMS) | — |
| LICENSE files | [`/releases/`](https://materios.fluxpointstudios.com/releases/) | — |
| Releases index | [`releases/`](https://materios.fluxpointstudios.com/releases/) (directory listing) | — |

**Always verify the sha256 of downloaded binaries against `SHA256SUMS`.**

> **The recommended path is `bootstrap-validator.sh`** — it handles binary download + verification + keystore seeding + systemd unit generation + validator startup in one command. See [Operator Guide → Permissioned Validator](operator-guide.md#permissioned-validator-non-spo) for the full walkthrough.

> **Data snapshot is required.** partner-chains-node v1.8.0 has a known historical-sync bug — fresh validators stall at block #0 (`Inherent error: Candidates inherent required`). Restore the **current** snapshot from [`operator-snapshots/preprod/latest.json`](https://materios.fluxpointstudios.com/operator-snapshots/preprod/latest.json) (published hourly, sha256-verified) into the rocksdb directory immediately after the bootstrap script finishes. The tarball contains only `data/chains/materios_preprod_v6/db/` — no keystore, no peer-id keys. The one-page recipe with the post-sync divergence self-check is [Current-Snapshot Bootstrap](current-snapshot-bootstrap.md).

> **arm64 / macOS:** the native binary is x86_64 Linux only. macOS operators either build from source (see [Operator Guide → Supported Environments](operator-guide.md#supported-environments) for the home-lab MacBook recipe) or run a Linux VM via UTM / Parallels / Hyper-V. Cert-daemon (attestor role) works natively on macOS via Docker Desktop.

> **glibc:** the binary is dynamically linked against glibc ≥ 2.38 (Ubuntu 22.04+ / Debian 12+ / RHEL 9+). Alpine / musl-libc systems are not supported.

## Hardware

Materios is lightweight. Most SPOs already run boxes that exceed these specs many times over for their Cardano node.

### SPO Validator and Permissioned Validator

The hardware spec is identical for both validator paths — same `materios-node` binary, same chain duties.

**Materios validator process alone** (the substrate node):

| Resource | Minimum | Recommended |
|---|---|---|
| CPU | 2 vCPU | 4 vCPU |
| RAM | 4 GB | 8 GB |
| Disk (Materios data) | 40 GB SSD | 100 GB SSD |

But you also need to run the **Cardano follower stack** on the same host — `cardano-node` (preprod) + `cardano-db-sync` + Postgres + Ogmios — because the Materios mainchain-follower reads committee state from your local db-sync via direct Postgres connection, not HTTP. That stack is **memory-heavy**. Realistic full-host requirements:

| Resource | Minimum (full stack) | Recommended (full stack) |
|---|---|---|
| CPU | 4 vCPU | 8 vCPU |
| RAM | **24 GB** | 32 GB |
| Disk | **150 GB NVMe/SSD** | 250 GB NVMe/SSD |
| Network | 10 Mbps, inbound TCP 30333 reachable | 100 Mbps |
| OS | Native Linux x86_64 + glibc ≥ 2.38 + systemd. Cloud VPS, bare-metal, or Hyper-V/VMware/VirtualBox VM. |

Per-component memory budget (approximate, preprod):

| Component | Typical RAM |
|---|---|
| `cardano-node` (preprod block producer / follower) | 8–12 GB |
| `cardano-db-sync` | 4–6 GB |
| Postgres (db-sync DB) | 4–8 GB (depends on `shared_buffers` / `work_mem`) |
| `ogmios` | 0.5 GB |
| `materios-node-spo` (the Materios validator) | 1–2 GB |
| OS + headroom | 2 GB |
| **Total realistic** | **~20–28 GB** |

Per-component disk budget (measured on the live preprod fleet, June 2026):

| Component | Disk |
|---|---|
| `cardano-node` chain db | ~18 GB |
| db-sync Postgres database | ~26 GB |
| `materios-node-spo` chain data | ~2 GB fresh, ~12 GB after weeks (cumulative) |
| Binaries, images, logs, snapshot staging | ~10–15 GB |
| **Total day-one** | **~60–70 GB** |

The 150 GB minimum is not padding: a db-sync re-sync (version bump, rollback recovery) transiently needs a **second full Postgres database** alongside the old one, Postgres needs vacuum/WAL slack, and everything above grows with chain history. A 100 GB disk starts ~65% full and hits the wall on the first re-sync event.

**A 12 GB host will not work** — Postgres + db-sync + cardano-node alone routinely exceed that under preprod load and the OOM killer will reap one of them, taking the validator with it. A 16 GB host is marginal: it can hold steady-state but tends to OOM during initial db-sync catch-up unless you tune `shared_buffers` down and add swap. **Treat 24 GB as the real floor; Hetzner CPX51 (16 vCPU / 32 GB / 360 GB NVMe, ~€42/mo) or CCX33 (8 dedicated vCPU / 32 GB / 240 GB) is the comfortable spec.** x86_64 only — the materios-node binary does not ship ARM builds, so skip the cheap Ampere/CAX tiers.

> **Cheaper option:** if you don't want to run the full Cardano stack yourself, use a hosted db-sync provider (TxPipe Dolos / Demeter.run / Blockfrost — check endpoint compatibility before committing). That lets you run materios-node-spo on a small host (4 vCPU / 8 GB / 40 GB SSD is plenty), with the Cardano-side state coming from the hosted provider over the network. Trade-off: dependency on a third party for L1 state.

> **Not supported:** WSL2, Docker Desktop running an amd64 Linux container on macOS or Windows, Alpine / musl-libc. The bootstrap script refuses to run on these. See [Operator Guide → Supported Environments](operator-guide.md#supported-environments) for the full list and recommended alternatives (UTM, Hyper-V, cloud VPS).

Observed preprod usage of the Materios validator process alone: ~800 MiB RAM, ~12 GB disk after several weeks, <1% average CPU. **Budget for growth** — chain history is cumulative.

The infrastructure differences between the two validator paths are:
- **SPO Validator:** runs cardano-node in *block-producer* mode (signing Cardano blocks), cardano-db-sync, Postgres, Ogmios, plus the Materios validator. Largest footprint; Cardano BP node is more memory-hungry than a passive follower.
- **Permissioned Validator:** runs cardano-node in *follower* mode (just consuming Cardano blocks), cardano-db-sync, Postgres, Ogmios, plus the Materios validator. Slightly lighter than SPO. **Or use a hosted db-sync provider** to avoid the Cardano stack entirely (recommended for low-budget operators).

### Separate requirement: cardano-db-sync

The Materios mainchain-follower reads Cardano preprod state from a **`cardano-db-sync` Postgres** that you provide. Choose one:

| Option | Good for | Notes |
|---|---|---|
| **Run your own** | Operators who already run cardano-db-sync for preprod. | Lowest ops risk. See [IOG docs](https://github.com/IntersectMBO/cardano-db-sync). |
| **Use a managed provider** | Operators who don't want another stateful service. | Public / paid options include [TxPipe Dolos](https://txpipe.io/), [Demeter.run](https://demeter.run/), [Blockfrost](https://blockfrost.io/) preprod (check endpoint compatibility before committing). |

The follower needs a direct Postgres connection to db-sync (not HTTP). If your provider only exposes HTTP/gRPC, it won't work — operators in this situation typically run db-sync themselves in a small VM.

Disk for db-sync is ~26 GB on preprod (measured June 2026) and grows with L1 history; keep room for a second full database during re-sync events.

### Ogmios

Required for **SPO registration** (`smart-contracts register`) and for the Materios node's Cardano RPC queries when running as an SPO. Either:
- run your own Ogmios pointed at your own cardano-node preprod, or
- use your db-sync provider's Ogmios endpoint if they expose one.

**Permissioned validators don't need Ogmios** unless they also plan to register as an SPO later. You can add it any time without restarting the node.

### Attestor

Standalone attestors only need the cert daemon (no Substrate node, no db-sync). One line to install:

| Resource | Minimum |
|---|---|
| CPU | 1 vCPU |
| RAM | 512 MB |
| Disk | 1 GB |
| Network | Outbound HTTPS + WSS only; no inbound ports |
| OS | Linux, macOS, or Windows with Docker |

See [Operator Guide → Attestor](operator-guide.md#attestor-permissionless).

## Network Ports

| Port | Direction | Purpose | Who opens it |
|---|---|---|---|
| **30333/tcp** | Inbound | libp2p P2P | SPO validators + full nodes. Must be reachable from `bootnode.materios.fluxpointstudios.com` (the public bootnode dials back). |
| **9945/tcp** | Localhost | JSON-RPC (HTTP + WS) for status queries | Everyone running a node; **never expose publicly** |
| **9615/tcp** | Localhost / LAN | Prometheus metrics | Optional |

A validator with RPC exposed to the public internet (`--rpc-external`, `--unsafe-rpc-external`, or `--rpc-methods unsafe` on a reachable address) is a misconfiguration and will be treated as such.

## Keys

An SPO validator holds three sets of keys. **All are generated locally — none of them ever leave your machine.**

| Key set | Purpose | Algorithm | Format |
|---|---|---|---|
| **Sidechain** | Partner-chain identity. Binds your SPO registration to a specific Materios account. | secp256k1 (ECDSA) | 64-char hex (compressed 33-byte pubkey → 66 hex) |
| **Aura** | Block authoring slot key. | sr25519 | SS58 + hex |
| **Grandpa** | Finality voting. | ed25519 | SS58 + hex |

The Partner Chains CLI (`partner-chains-node wizards generate-keys`, v1.8.0 binary) produces all three in one step plus a keystore directory the bootstrap script will pre-seed. See the [Operator Guide → Generate Materios validator keys](operator-guide.md#1-generate-materios-validator-keys). Note: the original IOG `input-output-hk/partner-chains` repo was [archived 2026-04-23](https://github.com/input-output-hk/partner-chains#warning-archived); upstream development moved into [`midnightntwrk/midnight-node`](https://github.com/midnightntwrk/midnight-node) under the new name `midnight-node-toolkit`. **The v1.8.0 binary remains the correct CLI for current Materios preprod** — it's the version Materios's runtime is built against. Don't substitute newer toolkit versions without coordination.

The bootstrap script (`bootstrap-validator.sh`) reads the JSON files in `~/materios-keys/` and pre-seeds the substrate keystore automatically. Manual `author_insertKey` is no longer required. You **do not** need to send the private keys to us — only the *public* portions, which we register on the Cardano-side permissioned-candidates list (or which you submit yourself in your SPO registration).

**Your sidechain private key authorizes your committee seat.** Losing it means your registration is effectively abandoned until you re-register with a new key. Back the mnemonic up offline.

## Runtime overrides — not needed on v6

v6 ships with the IDP-None fallback, Ariadne output deduplication, GRANDPA queue-depth guard (W-5 prevention), and other operational patches **baked into the on-chain runtime** (spec_version 211+). The `--wasm-runtime-overrides` flag is no longer required. Operators on v5 used to need a separate override WASM file; that path is retired.

Future runtime upgrades (spec_version bumps) are deployed via the standard Substrate `authorize_upgrade` + `apply_authorized_upgrade` flow, coordinated by the multisig sudo. As an operator you don't need to do anything for these — your node picks up the new runtime from the chain itself.

## Registration Requirements

### SPO Validator path

Before you can submit `smart-contracts register`, your Cardano preprod SPO needs:

- A registered stake pool (`stake-pool-registration-certificate` on-chain).
- **Active stake delegation** to that pool. Ariadne reads from the 2-epoch-stable snapshot, so your delegation must be live for at least **2 full preprod epochs (~10 days)** before you can be selected.
- Preprod tADA for:
  - Pool pledge + registration deposit (covered by normal SPO setup — not Materios-specific).
  - `smart-contracts register` transaction (~3 tADA).
  - `smart-contracts register` consumes one UTXO from your pool payment address as the `--registration-utxo`; leave a small unused UTXO there.

Detailed flow: [Operator Guide → SPO Validator](operator-guide.md#spo-validator).

### Permissioned Validator path

**No Cardano stake pool, no tADA, no 10-day stake-snapshot wait.** You generate Materios keys locally and send the *public* portions to Flux Point Studios. We submit a Cardano-side `upsert-permissioned-candidates` tx adding your sidechain pubkey to the partner-chains permissioned-candidates policy; you become eligible at the next stable mc_epoch boundary (typically ~3 hours, not 2 Cardano epochs).

What you submit to FPS:
- Sidechain (ECDSA) public key — 66-char hex.
- Aura (sr25519) public key — 64-char hex.
- Grandpa (ed25519) public key — 64-char hex.
- A label / contact (Discord handle is fine).

Private keys never leave your host. The pubkeys are public on-chain anyway once you're selected.

Detailed flow: [Operator Guide → Permissioned Validator](operator-guide.md#permissioned-validator-non-spo).

## Faucet Access

For the on-chain tx fees your Materios node needs once it's running (MATRA → MOTRA for the few extrinsics the operator-kit submits on your behalf), the preprod faucet auto-drips the node operator account when your cert daemon first registers.

No approval or whitelist — the gateway dedups on SS58, so each validator account gets exactly one drip.

## Chain Spec

Chain spec is served with CORS enabled so operators can curl it directly:

```
curl -o chain-spec.json https://materios.fluxpointstudios.com/releases/chain-spec-v6-raw.json
```

(The bootstrap script fetches this for you — only do this manually if you're not using `bootstrap-validator.sh`.)

Pin by verifying the genesis hash on first start:

```
expected_genesis=0x0e46e33f639a56cc8780fd871d9a15e16d99af248526f907cb560cb40849f7bf
```

`chain_getBlockHash(0)` on your synced node must match.

## Bootnodes

```
/dns4/bootnode.materios.fluxpointstudios.com/tcp/30333/p2p/12D3KooWPueKoxRAirTTKH4Y2qQAsJDegWMjS4k89Z7izCbZKgkM
```

One bootnode is enough to discover the rest. More bootnodes may be added as SPO validators come online; additions are published to this page.

## Monitoring

### Health

```bash
curl -s -X POST http://localhost:9945 \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"system_health"}'
```

Fields: `peers`, `isSyncing`, `shouldHavePeers`.

### Finality

Healthy = finalized trails best by ≤ 3 blocks. Any gap > 10 persisting for more than a minute is suspicious; check peer connectivity first and our [explorer](https://fluxpointstudios.com/materios/explorer) to see if the whole network is gapping or just you.

### Prometheus

`--prometheus-port 9615 --prometheus-external` exposes Substrate's standard metric set at `http://<node>:9615/metrics`. Bind it to LAN only unless you're happy to publish.

### Watchdog

We run a chain-level watchdog that pages Discord on finality stalls > 5 minutes. If something breaks at the network level, we'll know. For SPO-local alerting, wire Prometheus to your existing paging stack.

## Syncing

Preprod genesis → tip is fast (minutes on an SSD). During sync your node reports `isSyncing: true` and won't produce blocks even if it's in the committee. Insert Aura + Grandpa + sidechain keys **after** sync completes — not before.

## Platform Support

| Platform | Validator | Attestor | Path |
|---|---|---|---|
| Linux x86_64 (native, cloud VPS, bare metal) | ✅ | ✅ | `bootstrap-validator.sh` |
| Linux arm64 | ⚠️ | ✅ | Build validator from source — see [materios](https://github.com/Flux-Point-Studios/materios) `Dockerfile`. Attestor multi-arch image works as-is. |
| Hyper-V / VMware / VirtualBox VM running native Linux | ✅ | ✅ | Same as native Linux |
| macOS arm64 | 🛠 (manual) | ✅ | Validator: build from source + launchd; recipe is validated only on home LAN. Attestor: Docker Desktop. |
| macOS x86_64 (Rosetta) | ❌ | ✅ | Rosetta-translated libp2p drops peers; Attestor (multi-arch image) still works. |
| Windows + WSL2 | ❌ | ✅ | WSL2 NAT breaks libp2p P2P; bootstrap script refuses. Use Hyper-V Linux VM instead. Attestor (Docker Desktop on WSL2) still works. |
| Windows + Hyper-V Linux VM | ✅ | ✅ | Treat as native Linux |
| Alpine / musl-libc | ❌ | ⚠️ | Validator binary is glibc-linked; bootstrap refuses. |

See [Operator Guide → Supported Environments](operator-guide.md#supported-environments) for the rationale on each row. Pre-built arm64 binaries will be published when there's an SPO who needs them. Open an issue on [materios](https://github.com/Flux-Point-Studios/materios) if that's you.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Validator stuck at block #0 with `Inherent error: Candidates inherent required: committee needs to be stored one epoch in advance` | Known partner-chains-1.8.0 historical-sync bug | Apply the v6 data snapshot — see [Operator Guide → Permissioned Validator step 4](operator-guide.md#4-apply-the-v6-data-snapshot) |
| 0 peers immediately after start (peers connect briefly then drop) | Running on WSL2 / Docker Desktop on macOS / aggressive NAT | WSL2 NAT breaks libp2p — move to native Linux or a Hyper-V/UTM Linux VM. See [Operator Guide → Supported Environments](operator-guide.md#supported-environments). |
| 0 peers on a real Linux host | Firewall blocking inbound TCP 30333 | `sudo ufw allow 30333/tcp` + check NAT + cloud-provider security group; confirm with `nc -zv <your-public-ip> 30333` from a different network |
| `isSyncing` stuck | Wrong chain spec or all bootnodes unreachable | Verify genesis hash matches `0x0e46e33f639a56cc8780fd871d9a15e16d99af248526f907cb560cb40849f7bf`; re-check bootnode |
| Not producing blocks (but in committee) | Keystore not seeded, or pre-seeded against the wrong chain ID | Confirm `~/materios-preprod/data/chains/materios_preprod_v6/keystore/` has 3 files (one per scheme). Re-run `bootstrap-validator.sh` if not. |
| Cert-daemon `Heartbeat rejected: 403 {"error":"Validator not registered"}` | Gateway's api_keys registry doesn't have your SS58 (e.g. after a chain reset) | Contact Flux Point Studios to re-register your SS58 |
| Finality gap growing | Your validator's GRANDPA vote isn't reaching peers | Check outbound network, clock skew (`timedatectl`), db-sync freshness |
| `Inability to pay some fees` | Account out of MOTRA | Wait for a few blocks (MATRA auto-mints MOTRA); or request a faucet drip |
| `State already discarded` in cert-daemon logs | Cert-daemon cursor is older than the node's pruning window | Wipe `daemon-state.json`, restart daemon |

## Source

Chain + runtime: [`Flux-Point-Studios/materios`](https://github.com/Flux-Point-Studios/materios)
Operator kit (attestor, monitoring): [`Flux-Point-Studios/materios-operator-kit`](https://github.com/Flux-Point-Studios/materios-operator-kit)

Both are public. The same binary you download from `/releases/` is built from the `main` branch.
