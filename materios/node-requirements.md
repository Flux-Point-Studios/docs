---
description: Hardware, software, keys, and infrastructure needed to run a Materios SPO validator, attestor, or full node on the preprod network.
---

# Node Requirements

Materios is a Cardano Partner Chain. Committee selection uses Ariadne: seats are allocated to Cardano stake pool operators weighted by preprod ada delegation. There is **no permissioned allowlist** — join by registering as an SPO on Cardano preprod and your stake makes you eligible.

There are three roles:

| Role | What it does | Requires Cardano SPO? |
|------|---------|---|
| **SPO Validator** | Produces blocks, votes on finality, signs attestations, earns tMATRA. | **Yes** — register on Cardano preprod via `partner-chains-node smart-contracts register`. |
| **Attestor** | Verifies blobs and signs attestations. Earns 10 tMATRA per certified receipt. | No. Permissionless, one-liner install. |
| **Full Node** | RPC + archive / read-only peer. | No. |

See the [Operator Guide](operator-guide.md) for end-to-end setup flows.

## Current Network

| Field | Value |
|---|---|
| Chain | `Materios Preprod v5` |
| Chain ID | `materios_preprod_v5` |
| Genesis hash | `0xbc0531cb311281565036fb397a376f0e0fa37005589655f97a7924b2729a164c` |
| spec_version | 201 |
| Block time | 6 seconds |
| Session length | 60 blocks (~6 minutes) |
| Cardano L1 | Preprod testnet |
| Main-chain epoch length | 5 days |
| D-parameter | `(3, 2)` — 3 permissioned + 2 registered SPO seats per committee |
| Public RPC (WSS) | `wss://materios.fluxpointstudios.com/preprod-rpc` |
| Explorer | [fluxpointstudios.com/materios/explorer](https://fluxpointstudios.com/materios/explorer) |

The permissioned seats are held by Flux Point Studios. The **2 registered seats rotate** among all SPOs who have registered via the smart contracts. Selection probability is weighted by each pool's active preprod-ada stake.

## Static Asset Distribution

Everything an operator needs to download lives under `materios.fluxpointstudios.com`:

| Asset | URL | sha256 |
|---|---|---|
| Chain spec (raw JSON) | [`chain-spec-v5-raw.json`](https://materios.fluxpointstudios.com/chain-spec-v5-raw.json) | — |
| Node binary (x86_64 Linux) | [`releases/materios-node-v5-x86_64-linux`](https://materios.fluxpointstudios.com/releases/materios-node-v5-x86_64-linux) | `b55bfc51…e8119c` |
| Runtime override WASM | [`releases/materios_runtime.compact.compressed.wasm`](https://materios.fluxpointstudios.com/releases/materios_runtime.compact.compressed.wasm) | `df135633…5bed7` |
| SHA256SUMS manifest | [`releases/SHA256SUMS`](https://materios.fluxpointstudios.com/releases/SHA256SUMS) | — |
| Releases index | [`releases/`](https://materios.fluxpointstudios.com/releases/) (directory listing) | — |

**Always verify the sha256 of downloaded binaries and WASM against `SHA256SUMS`.**

> **arm64 / macOS:** the native binary is x86_64 Linux only. ARM operators either build from source (see `Dockerfile` in the [materios](https://github.com/Flux-Point-Studios/materios) repo) or run the x86_64 binary under emulation (Rosetta / qemu).

> **glibc:** the binary is dynamically linked against glibc ≥ 2.35 (Ubuntu 22.04+ / Debian 12+ / RHEL 9+). On older distros the loader will refuse to start — upgrade or build from source.

## Hardware

Materios is lightweight. Most SPOs already run boxes that exceed these specs many times over for their Cardano node.

### SPO Validator

| Resource | Minimum | Recommended |
|---|---|---|
| CPU | 2 vCPU | 4 vCPU |
| RAM | 2 GB | 4 GB |
| Disk (Materios data) | 50 GB SSD | 100 GB SSD |
| Network | 10 Mbps, static IPv4 | 100 Mbps |
| OS | Linux x86_64 (Ubuntu 22.04+, Debian 12+, RHEL 9+) |

Observed preprod usage: ~800 MiB RAM, ~12 GB disk after several weeks, <1% average CPU. **Budget for growth** — chain history is cumulative.

### Separate requirement: cardano-db-sync

The Materios mainchain-follower reads Cardano preprod state from a **`cardano-db-sync` Postgres** that you provide. Choose one:

| Option | Good for | Notes |
|---|---|---|
| **Run your own** | Operators who already run cardano-db-sync for preprod. | Lowest ops risk. See [IOG docs](https://github.com/IntersectMBO/cardano-db-sync). |
| **Use a managed provider** | Operators who don't want another stateful service. | Public / paid options include [TxPipe Dolos](https://txpipe.io/), [Demeter.run](https://demeter.run/), [Blockfrost](https://blockfrost.io/) preprod (check endpoint compatibility before committing). |

The follower needs a direct Postgres connection to db-sync (not HTTP). If your provider only exposes HTTP/gRPC, it won't work — operators in this situation typically run db-sync themselves in a small VM.

Disk for db-sync is ~25 GB on preprod and grows with L1 history.

### Ogmios

Required for SPO registration (`smart-contracts register`) and for the Materios node's Cardano RPC queries. Either:
- run your own Ogmios pointed at your own cardano-node preprod, or
- use your db-sync provider's Ogmios endpoint if they expose one.

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
| **30333/tcp** | Inbound | libp2p P2P | SPO validators + full nodes |
| **9944/tcp** | Localhost | JSON-RPC (HTTP + WS) for key insertion and local queries | Everyone running a node; **never expose publicly** |
| **9615/tcp** | Localhost / LAN | Prometheus metrics | Optional |

A validator with RPC exposed to the public internet (`--rpc-external`, `--unsafe-rpc-external`, or `--rpc-methods unsafe` on a reachable address) is a misconfiguration and will be treated as such.

## Keys

An SPO validator holds three sets of keys. **All are generated locally — none of them ever leave your machine.**

| Key set | Purpose | Algorithm | Format |
|---|---|---|---|
| **Sidechain** | Partner-chain identity. Binds your SPO registration to a specific Materios account. | secp256k1 (ECDSA) | 64-char hex (compressed 33-byte pubkey → 66 hex) |
| **Aura** | Block authoring slot key. | sr25519 | SS58 + hex |
| **Grandpa** | Finality voting. | ed25519 | SS58 + hex |

The IOG CLI (`partner-chains-node key generate`) produces all three in one step. See the [Operator Guide → Generate Materios keys](operator-guide.md#2-generate-materios-keys).

Keys go into the node's keystore via the `author_insertKey` RPC **after** the node starts. You **do not** need to send the keys to us — only the *public* portions are referenced in your on-chain SPO registration.

**Your sidechain private key authorizes your committee seat.** Losing it means your registration is effectively abandoned until you re-register with a new key. Back the mnemonic up offline.

## Runtime Overrides

Materios preprod v5 requires a WASM runtime override. The override is loaded from disk at node start via `--wasm-runtime-overrides`. This is a normal Substrate flag — it does **not** modify the on-chain runtime, and every validator running the override must have an identical copy.

Current override ships the following local patches on top of the genesis runtime:

1. **IOG IDP-None fallback** — when the mainchain follower cannot compute a committee for the current session (happens at some main-chain epoch boundaries), the pallet reuses the previous committee instead of panicking. Upstream issue tracked at [IOG partner-chains](https://github.com/input-output-hk/partner-chains).
2. **Ariadne output deduplication** — the with-replacement weighted-random sampler can emit duplicate seats for the same SPO; we collapse duplicates before the set is passed to GRANDPA. Prevents finality stalls when the duplicated validator is offline.

Runtime overrides will be removed as these fixes land upstream. We re-publish the override each time the override content changes; **pin by sha256** in your systemd unit so you know when to update.

## Registration Requirements

Before you can submit `smart-contracts register`, your Cardano preprod SPO needs:

- A registered stake pool (`stake-pool-registration-certificate` on-chain).
- **Active stake delegation** to that pool. Ariadne reads from the 2-epoch-stable snapshot, so your delegation must be live for at least **2 full preprod epochs (~10 days)** before you can be selected.
- Preprod tADA for:
  - Pool pledge + registration deposit (covered by normal SPO setup — not Materios-specific).
  - `smart-contracts register` transaction (~3 tADA).
  - `smart-contracts register` consumes one UTXO from your pool payment address as the `--registration-utxo`; leave a small unused UTXO there.

Detailed flow: [Operator Guide → SPO Validator](operator-guide.md#spo-validator).

## Faucet Access

For the on-chain tx fees your Materios node needs once it's running (MATRA → MOTRA for the few extrinsics the operator-kit submits on your behalf), the preprod faucet auto-drips the node operator account when your cert daemon first registers.

No approval or whitelist — the gateway dedups on SS58, so each validator account gets exactly one drip.

## Chain Spec

Chain spec is served with CORS enabled so operators can curl it directly:

```
curl -o chain-spec.json https://materios.fluxpointstudios.com/chain-spec-v5-raw.json
```

Pin by verifying the genesis hash on first start:

```
expected_genesis=0xbc0531cb311281565036fb397a376f0e0fa37005589655f97a7924b2729a164c
```

`chain_getBlockHash(0)` on your synced node must match.

## Bootnodes

```
/ip4/166.70.250.197/tcp/30333/p2p/12D3KooWPueKoxRAirTTKH4Y2qQAsJDegWMjS4k89Z7izCbZKgkM
```

One bootnode is enough to discover the rest. More bootnodes may be added as SPO validators come online; additions are published to this page.

## Monitoring

### Health

```bash
curl -s -X POST http://localhost:9944 \
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

| Platform | Supported | Path |
|---|---|---|
| Linux x86_64 | ✅ | Native binary (`releases/materios-node-v5-x86_64-linux`) |
| Linux arm64 | ⚠️ | Build from source — see [materios](https://github.com/Flux-Point-Studios/materios) `Dockerfile` |
| macOS arm64 | ⚠️ | Build from source; cert-daemon for attestation works natively |
| macOS x86_64 | ⚠️ | Run x86_64 binary under Rosetta; unsupported in production |
| Windows | ⚠️ | WSL2 only; treat as Linux |

Pre-built arm64 + macOS binaries will be published when there's an SPO who needs them. Open an issue on [materios](https://github.com/Flux-Point-Studios/materios) if that's you.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| 0 peers | Firewall blocking 30333 | `sudo ufw allow 30333/tcp` + check NAT |
| `isSyncing` stuck | Wrong chain spec or all bootnodes unreachable | Verify genesis hash; re-check bootnode list |
| Not producing blocks (but in committee) | Keys not inserted, or inserted after sync | Re-insert keys via `author_insertKey`, restart |
| Panic at main-chain epoch boundary | Missing or stale runtime override | Re-download the WASM, verify sha256, restart |
| Finality gap growing | Your validator's GRANDPA vote isn't reaching peers | Check outbound network, clock skew (`timedatectl`), db-sync freshness |
| `Inability to pay some fees` | Account out of MOTRA | Wait for a few blocks (MATRA auto-mints MOTRA); or request a faucet drip |
| `State already discarded` in cert-daemon logs | Cert-daemon cursor is older than the node's pruning window | Wipe `daemon-state.json`, restart daemon |

## Source

Chain + runtime: [`Flux-Point-Studios/materios`](https://github.com/Flux-Point-Studios/materios)
Operator kit (attestor, monitoring): [`Flux-Point-Studios/materios-operator-kit`](https://github.com/Flux-Point-Studios/materios-operator-kit)

Both are public. The same binary you download from `/releases/` is built from the `main` branch.
