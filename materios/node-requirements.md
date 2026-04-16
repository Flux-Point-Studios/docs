---
description: Hardware, software, and configuration for running a Materios node or attestor
---

# Node Requirements

This page covers what you need to participate in the Materios network. There are three roles:

| Role | Purpose | Cardano stack? | Approval |
|------|---------|----------------|----------|
| **Attestor** | Blob verification + attestation rewards | **No** — just cert-daemon against our public RPC | **No approval needed** |
| **Full Validator (SPO)** | Block production + finality + attestation | **Yes** — your own cardano-node + cardano-db-sync + Postgres | No approval needed |
| **Full Node / RPC mirror** | Sync chain + serve RPC queries | **Yes, currently** — see note below | None |

> **Why Cardano?** Materios v3 is an IOG partner-chain. Committee selection and the D-parameter are read from Cardano preprod on-chain state. Any Materios node that verifies blocks — validator or not — needs to re-derive those inherent values, which requires a reachable `cardano-db-sync` Postgres endpoint.
>
> **If you don't want Cardano infra, run attestor mode.** It's the supported path for operators who just want to earn rewards without running a second blockchain stack.

## Choosing your role

```
Do you want to run a cert-daemon + earn attestation rewards?
│
├── Yes, but I don't want to run a Cardano node ────────→ Attestor (recommended)
│
├── Yes, and I already run a Cardano SPO (or don't mind
│   running cardano-node + db-sync + Postgres) ─────────→ Full Validator
│
└── I need to sync Materios state for a dApp, explorer,
    analytics, or RPC mirror — not participating in
    consensus ──────────────────────────────────────────→ Full Node
                                                           (requires db-sync
                                                            same as validators,
                                                            currently)
```

## Hardware Requirements

### Validator Node

These numbers are **just for the Materios node itself**. Add the Cardano stack requirements below if you don't already operate one.

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| **CPU** | 2 vCPUs | 4 vCPUs |
| **RAM** | 1 GB | 2 GB |
| **Disk** | 20 GB SSD | 50 GB SSD |
| **Network** | 10 Mbps, static IP | 100 Mbps, static IP |
| **OS** | Linux (x86\_64) | Ubuntu 22.04+ or Debian 12+ |

Actual observed usage on the preprod network: ~800 MiB RAM, <1% CPU, ~12 GB disk after several weeks of operation. Requirements will grow as chain history accumulates.

### Cardano Stack (for full validators + full nodes)

Materios reads Cardano preprod state via cardano-db-sync. You need these services reachable from your Materios node:

| Service | Purpose | Footprint (preprod) |
|---------|---------|---------------------|
| **cardano-node** (preprod) | Follow the Cardano chain | ~15 GB disk, ~1 GB RAM |
| **cardano-db-sync** | Index chain state into Postgres | ~15 GB disk, ~1 GB RAM |
| **Postgres 16** | Storage backend | <1 GB RAM |

Initial sync takes several hours on preprod (can be accelerated with a Mithril snapshot). Reference Docker Compose:

```yaml
services:
  cardano-node-preprod:
    image: ghcr.io/intersectmbo/cardano-node:10.1.4
    restart: unless-stopped
    environment: { NETWORK: preprod }
    volumes:
      - node-data:/data
      - node-ipc:/ipc

  postgres-preprod:
    image: postgres:16
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: change-me
      POSTGRES_DB: cexplorer
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5433:5432"

  db-sync-preprod:
    image: ghcr.io/intersectmbo/cardano-db-sync:13.6.0.4
    restart: unless-stopped
    depends_on:
      postgres-preprod: { condition: service_healthy }
    environment:
      NETWORK: preprod
      POSTGRES_HOST: postgres-preprod
      POSTGRES_PORT: 5432
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: change-me
      POSTGRES_DB: cexplorer
    volumes:
      - db-sync-data:/var/lib/cdbsync
      - node-ipc:/node-ipc:ro

volumes: { node-data: , node-ipc: , postgres-data: , db-sync-data: }
```

After this is running, pass its connection string to the Materios installer:

```bash
export DB_SYNC_POSTGRES_CONNECTION_STRING='postgres://postgres:change-me@localhost:5433/cexplorer'
curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install.sh | bash
```

### Full Node / Archive

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| **CPU** | 2 vCPUs | 4 vCPUs |
| **RAM** | 2 GB | 4 GB |
| **Disk** | 50 GB SSD | 100 GB SSD |
| **Network** | 10 Mbps | 100 Mbps |

Archive nodes retain all historical state and serve RPC queries. They require more disk and memory than validators.

### Attestor (Cert Daemon Only)

Attestors run a cert daemon without a full Substrate node. They connect to the public Materios RPC endpoint (`wss://materios.fluxpointstudios.com/preprod-rpc`) and earn attestation rewards (10 tMATRA per certified receipt). **No approval required.**

| Resource | Minimum |
|----------|---------|
| **CPU** | 1 vCPU |
| **RAM** | 512 MB |
| **Disk** | 1 GB |
| **Network** | Outbound HTTPS + WSS only (no ports to open) |
| **OS** | Linux, macOS, or Windows with Docker |

One-command install:

```bash
curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install.sh | bash -s -- --mode attestor
```

Or download a platform-specific installer (no terminal needed):

| Platform | Download |
|----------|----------|
| **macOS** | [install-macos.command](https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install-macos.command) — double-click to run |
| **Windows** | [install-windows.bat](https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install-windows.bat) — double-click to run |
| **Linux** | [install-linux.sh](https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install-linux.sh) — `chmod +x && ./install-linux.sh` |

See the [Operator Guide](operator-guide.md#attestor-permissionless) for full details.

## Software

### Container Image

The Materios node is distributed as a Docker image built from the monorepo.

```bash
docker pull ghcr.io/flux-point-studios/materios-node:v3
```

The image is publicly available on GHCR. No need to build from source unless you want to audit the code.

Image size: ~224 MB uncompressed (~56 MB compressed).

### Runtime

- **spec\_name**: `materios`
- **spec\_version**: v3
- **Chain ID**: `materios_preprod`
- **Block time**: 6 seconds

## Network Ports

| Port | Protocol | Purpose | Required |
|------|----------|---------|----------|
| **30333** | TCP | P2P (libp2p) | Yes (inbound + outbound) |
| **9944** | TCP | JSON-RPC (HTTP + WebSocket) | Validators: localhost only. Full nodes: as needed. |
| **9615** | TCP | Prometheus metrics | Optional |

**Security note**: Validators should bind RPC to `127.0.0.1` only. Never expose `--unsafe-rpc-external` or `--rpc-methods unsafe` on a public-facing validator.

## Configuration

### Validator Node

```bash
materios-node \
  --chain /path/to/chain-spec-raw.json \
  --base-path /data/materios \
  --validator \
  --name your-node-name \
  --rpc-port 9944 \
  --port 30333 \
  --prometheus-port 9615 \
  --prometheus-external \
  --node-key <your-32-byte-hex-node-key>
```

**Key flags**:

| Flag | Purpose |
|------|---------|
| `--chain /path/to/chain-spec-raw.json` | Use the Materios local chain spec |
| `--validator` | Enable block production and finality voting |
| `--base-path` | Where chain data is stored |
| `--node-key` | Persistent P2P identity (32 hex bytes). Generate with `subkey generate-node-key` |
| `--name` | Human-readable name shown in telemetry |

### Full Node (Non-Validator)

Full nodes omit `--validator` but **still need a reachable cardano-db-sync Postgres endpoint** because the IOG session-validator-management pallet verifies inherent data on every block import. A node with no mainchain access cannot sync past the first session rotation.

```bash
materios-node \
  --chain /path/to/chain-spec-raw.json \
  --base-path /data/materios \
  --name your-node-name \
  --rpc-port 9944 \
  --rpc-cors all \
  --unsafe-rpc-external \
  --port 30333 \
  --pruning archive
```

Required environment variables (same as validator mode):

```bash
export DB_SYNC_POSTGRES_CONNECTION_STRING='postgres://user:pass@host:5432/cexplorer'
export MC__FIRST_EPOCH_TIMESTAMP_MILLIS=1654041600000
export MC__EPOCH_DURATION_MILLIS=432000000
export MC__FIRST_EPOCH_NUMBER=0
export MC__FIRST_SLOT_NUMBER=0
export CARDANO_SECURITY_PARAMETER=2160
export CARDANO_ACTIVE_SLOTS_COEFF=0.05
export PARTNER_CHAIN_GENESIS_UTXO='0bacdb7e50ba61a1f9e28007a4f9543fa0e8e31ce10027b2f1dda8ab3438d388#0'
export COMMITTEE_CANDIDATE_ADDRESS='addr_test1wzr6en3y43437qps5wscegufxw0euspmy0c3976mjm95j0cwuvezm'
export D_PARAMETER_POLICY_ID=0x7f57bb675447c65ba0d68270a6b9b93aecc8dfdacaa3aa8cd081f9f3
export PERMISSIONED_CANDIDATES_POLICY_ID=0x70cd1c6fbbbd7b1e855f589abd842f433ec0d7b46c7a9e437194e931
```

> **Don't want to run a Cardano stack?** Today this isn't cleanly supported for full nodes. The only path that works without Cardano infra is **attestor mode** (cert-daemon only, no Materios node, reads chain state from our public RPC). If you specifically need a local Materios RPC mirror for a dApp or explorer backend and can't run Cardano, open an issue — we're tracking interest in a shared/public mainchain-data endpoint.

### Environment Variables

```bash
RUST_LOG=info,materios=debug
```

### Bootnodes

New nodes need at least one bootnode to discover peers. The preprod chain uses the following bootnodes:

```
/ip4/166.70.250.197/tcp/30333/p2p/12D3KooWPueKoxRAirTTKH4Y2qQAsJDegWMjS4k89Z7izCbZKgkM
```

Add these to your node's `--bootnodes` flag. The install script configures this automatically.

## Session Keys

Validators require two session keys:

| Key Type | Scheme | Purpose |
|----------|--------|---------|
| **Aura** | sr25519 | Block authoring |
| **Grandpa** | ed25519 | Finality voting |

### Generating Keys

```bash
# Generate Aura key
subkey generate --scheme sr25519

# Generate Grandpa key
subkey generate --scheme ed25519
```

### Inserting Keys

Keys are inserted into the running node via RPC:

```bash
# Insert Aura key
curl -X POST http://localhost:9944 \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "author_insertKey",
    "params": ["aura", "<mnemonic>", "<public-key-hex>"],
    "id": 1
  }'

# Insert Grandpa key
curl -X POST http://localhost:9944 \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "author_insertKey",
    "params": ["gran", "<mnemonic>", "<public-key-hex>"],
    "id": 1
  }'
```

Keys persist to the node's keystore on disk. After insertion, restart the node for Grandpa keys to take effect.

**Security**: Keep mnemonics secure. Never share them. The `author_insertKey` RPC should only be accessible on localhost.

## Chain Spec

The chain specification file defines genesis state, initial authorities, and network parameters. The preprod chain spec is available at [`chain-spec/preprod-chain-spec-raw.json`](https://github.com/Flux-Point-Studios/materios/blob/main/chain-spec/preprod-chain-spec-raw.json).

To verify you have the correct chain spec:

```bash
# Expected preprod genesis hash
0x105fed4310b56550d3646a13d7a6f4b69ab82f1c1269a6c732948a2a260b1360
```

> **Note**: No WASM overrides are needed for the preprod chain. All runtime fixes are baked into the genesis. If you previously ran a preview/staging node with WASM overrides, you can safely remove them.

## Monitoring

### Health Check

```bash
curl -s http://localhost:9944 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"system_health","params":[],"id":1}'
```

Response includes `peers` (connected peer count), `isSyncing` (whether the node is catching up), and `shouldHavePeers` (whether the node expects peers).

### Prometheus Metrics

If `--prometheus-port 9615` is set, metrics are available at `http://localhost:9615/metrics` for integration with Grafana or other monitoring tools.

### Key Metrics to Watch

- **Block height**: Should advance every 6 seconds
- **Peer count**: Validators should have at least 2 peers
- **Finalized block**: Should trail head by no more than a few blocks
- **Memory usage**: Typical ~800 MiB, investigate if >2 GB

## Docker Compose Example

```yaml
version: "3.8"
services:
  materios-node:
    image: ghcr.io/flux-point-studios/materios-node:v3
    container_name: materios-node
    restart: unless-stopped
    user: "1000:1000"
    ports:
      - "30333:30333"
      - "127.0.0.1:9944:9944"
      - "9615:9615"
    volumes:
      - materios-data:/data/materios
    environment:
      - RUST_LOG=info,materios=debug
    command:
      - --chain
      - /data/materios/preprod-chain-spec-raw.json
      - --base-path
      - /data/materios
      - --validator
      - --name
      - my-materios-validator
      - --rpc-port
      - "9944"
      - --port
      - "30333"
      - --prometheus-port
      - "9615"
      - --prometheus-external
      - --node-key
      - "<your-32-byte-hex-node-key>"

volumes:
  materios-data:
```

**Note**: Replace `<your-32-byte-hex-node-key>` with your actual node key. For production, consider passing sensitive values via environment variables or Docker secrets rather than command-line arguments.

## Syncing a New Node

New nodes sync from genesis by default. The preprod network is young, so a full sync completes in minutes. As the chain grows, snapshot-based syncing may be offered.

During sync, the node will show `isSyncing: true` in the health check. Wait for sync to complete before inserting validator keys.

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| 0 peers | Firewall blocking port 30333 | Open TCP 30333 inbound |
| `isSyncing` stuck | Wrong chain spec or no bootnodes | Verify genesis hash, add bootnodes |
| Not producing blocks | Keys not inserted or wrong key type | Re-insert keys, restart node |
| Finality stalled | Grandpa key mismatch or <2/3 validators online | Check key insertion, check peer connectivity |
| High memory (>4 GB) | Possible leak or archive mode on limited RAM | Restart node, check `--pruning` setting |
| `missing field db_sync_postgres_connection_string` | Validator mode but no Cardano stack | Set `DB_SYNC_POSTGRES_CONNECTION_STRING` or switch to `--mode attestor` |
| `Stable block not found at <timestamp>` | db-sync not caught up to current Cardano tip | Wait for db-sync to finish syncing preprod (hours from genesis, minutes from Mithril snapshot) |
| `NetworkKeyNotFound` | First-time run without `--unsafe-force-node-key-generation` | Installer handles this; if running the binary directly, pass the flag once |
| `no matching manifest for linux/arm64/v8` | Apple Silicon pulling without platform flag | `docker pull --platform linux/amd64 ghcr.io/flux-point-studios/materios-node:v3` |
| Heartbeats show stale `best_block` from old chain | cert-daemon carried over `daemon-state.json` from a prior chain reset | Re-run the installer (it wipes volumes on re-install now) |

## Platform Support

The Materios node Docker image is currently **x86\_64 (amd64) only**. ARM64 hosts run it under emulation:

| Platform | Architecture | How to Run |
|----------|-------------|------------|
| **Linux (x86\_64)** | amd64 | Native — `docker pull ghcr.io/flux-point-studios/materios-node:v3` |
| **Linux (ARM64)** | arm64 | Requires `qemu-user-static` for cross-arch, or wait for native build |
| **macOS (Apple Silicon)** | arm64 | Docker Desktop runs x86\_64 under Rosetta — pull with `--platform linux/amd64`. The installer adds this flag automatically. |
| **macOS (Intel)** | amd64 | Native — Docker Desktop, then same command |
| **Windows** | amd64 | Docker Desktop + WSL2 (see below) |

The installer detects your architecture and passes the right `--platform` flag to `docker pull`. If you're pulling manually on Apple Silicon, use:

```bash
docker pull --platform linux/amd64 ghcr.io/flux-point-studios/materios-node:v3
```

A native arm64 image is on the roadmap.

### Windows Setup (Detailed)

Windows requires Docker Desktop with WSL2. Here's the full setup:

1. **Install WSL2** (if not already enabled):
   ```powershell
   # Run in PowerShell as Administrator
   wsl --install
   # Restart your computer when prompted
   ```

2. **Install Docker Desktop**: Download from [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/). During installation, ensure **"Use WSL 2 instead of Hyper-V"** is checked.

3. **Open Ubuntu terminal**: After WSL installs, search for "Ubuntu" in the Start menu and open it. This is where you'll run all commands.

4. **Verify Docker works in WSL**:
   ```bash
   docker --version        # Should show Docker Engine 20+
   docker compose version  # Should show Compose v2
   ```

5. **Open port 30333** (for validators): Windows Firewall > Advanced Settings > Inbound Rules > New Rule > Port > TCP > 30333 > Allow.

6. **Run the installer** (inside the Ubuntu terminal, NOT PowerShell):
   ```bash
   curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install.sh | bash -s -- --label my-validator
   ```

> **Important:** All Materios commands must be run inside the WSL Ubuntu terminal. The install script is a bash script and does not run natively in PowerShell or Command Prompt.

### Raspberry Pi / ARM SBCs

The arm64 image runs on Raspberry Pi 4/5 and other ARM64 single-board computers with at least 2 GB RAM. Performance is acceptable for validator operation given the lightweight chain requirements.

### Chain Spec

The installer automatically downloads the chain spec. If you need it manually:

```bash
curl -sSLO https://materios.fluxpointstudios.com/chain-spec-v3-raw.json
```
