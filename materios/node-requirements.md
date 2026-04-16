---
description: Hardware, software, and configuration for running a Materios node or attestor
---

# Node Requirements

This page covers what you need to participate in the Materios network. There are three roles:

| Role | Purpose | Approval |
|------|---------|----------|
| **Full Validator** | Block production + finality + attestation | No approval needed |
| **Attestor** | Blob verification + attestation rewards | **No approval needed** |
| **Full Node** | Sync chain + serve RPC queries | None |

## Hardware Requirements

### Validator Node

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| **CPU** | 2 vCPUs | 4 vCPUs |
| **RAM** | 1 GB | 2 GB |
| **Disk** | 20 GB SSD | 50 GB SSD |
| **Network** | 10 Mbps, static IP | 100 Mbps, static IP |
| **OS** | Linux (x86\_64) | Ubuntu 22.04+ or Debian 12+ |

Actual observed usage on the preprod network: ~800 MiB RAM, <1% CPU, ~12 GB disk after several weeks of operation. Requirements will grow as chain history accumulates.

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

Full nodes omit `--validator` and can safely expose RPC externally with `--rpc-cors` for serving queries.

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

## Platform Support

The Materios node Docker image supports multiple platforms:

| Platform | Architecture | How to Run |
|----------|-------------|------------|
| **Linux (x86_64)** | amd64 | `docker pull ghcr.io/flux-point-studios/materios-node:v3` |
| **Linux (ARM64)** | arm64 | Same command — Docker selects the right image automatically |
| **macOS (Apple Silicon)** | arm64 | Install [Docker Desktop](https://www.docker.com/products/docker-desktop/), then same command |
| **macOS (Intel)** | amd64 | Install Docker Desktop, then same command |
| **Windows** | amd64/arm64 | Install Docker Desktop with WSL2 backend (see below) |

Docker automatically pulls the correct architecture for your machine. No special flags needed.

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
