---
description: Hardware, software, and configuration for running a Materios node
---

# Node Requirements

This page covers what you need to run a Materios node, whether as a **validator** (block producer + finalizer) or a **full node** (sync and serve RPC).

## Hardware Requirements

### Validator Node

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| **CPU** | 2 vCPUs | 4 vCPUs |
| **RAM** | 1 GB | 2 GB |
| **Disk** | 20 GB SSD | 50 GB SSD |
| **Network** | 10 Mbps, static IP | 100 Mbps, static IP |
| **OS** | Linux (x86\_64) | Ubuntu 22.04+ or Debian 12+ |

Actual observed usage on the staging network: ~800 MiB RAM, <1% CPU, ~12 GB disk after several weeks of operation. Requirements will grow as chain history accumulates.

### Full Node / Archive

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| **CPU** | 2 vCPUs | 4 vCPUs |
| **RAM** | 2 GB | 4 GB |
| **Disk** | 50 GB SSD | 100 GB SSD |
| **Network** | 10 Mbps | 100 Mbps |

Archive nodes retain all historical state and serve RPC queries. They require more disk and memory than validators.

### External Operator (Cert Daemon Only)

External attestation committee members run only the cert daemon, not a full Substrate node. Requirements are minimal:

| Resource | Minimum |
|----------|---------|
| **CPU** | 1 vCPU |
| **RAM** | 512 MB |
| **Disk** | 1 GB |
| **Network** | Outbound HTTPS + WSS |

## Software

### Container Image

The Materios node is distributed as a Docker image built from the monorepo.

> **Note**: A pre-built container image is not yet publicly available. For now, you must build from source (see below). A published image on GHCR will be announced once the repository is public.

```bash
docker build -t materios-node:latest -f partnerchain/Dockerfile .
```

**Build toolchain** (if building from source):
- Rust 1.88.0 with `wasm32-unknown-unknown` target
- System packages: `protobuf-compiler`, `clang`, `libclang-dev`, `libssl-dev`, `pkg-config`, `make`, `cmake`
- Polkadot SDK: `polkadot-stable2409-5`

The resulting image is ~224 MB uncompressed (~56 MB compressed).

### Runtime

- **spec\_name**: `materios`
- **spec\_version**: 105
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
  --chain staging \
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
| `--chain staging` | Use the Materios staging chain spec |
| `--validator` | Enable block production and finality voting |
| `--base-path` | Where chain data is stored |
| `--node-key` | Persistent P2P identity (32 hex bytes). Generate with `subkey generate-node-key` |
| `--name` | Human-readable name shown in telemetry |

### Full Node (Non-Validator)

```bash
materios-node \
  --chain staging \
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

New nodes need at least one bootnode to discover peers. Bootnodes will be provided during the onboarding process. Contact the Flux Point Studios team for current bootnode addresses.

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

The chain specification file defines genesis state, initial authorities, and network parameters. The raw chain spec for the staging network is included in the repository at `ops/chain-spec-raw.json`.

To verify you have the correct chain spec:

```bash
# Expected genesis hash
8f6e531be80341a12a0ae1b04484770fcaa797bb49dcc1cc9e79788f770a41b3
```

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
    image: materios-node:latest
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
      - staging
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

New nodes sync from genesis by default. The staging network is young, so a full sync completes in minutes. As the chain grows, snapshot-based syncing may be offered.

During sync, the node will show `isSyncing: true` in the health check. Wait for sync to complete before inserting validator keys.

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| 0 peers | Firewall blocking port 30333 | Open TCP 30333 inbound |
| `isSyncing` stuck | Wrong chain spec or no bootnodes | Verify genesis hash, add bootnodes |
| Not producing blocks | Keys not inserted or wrong key type | Re-insert keys, restart node |
| Finality stalled | Grandpa key mismatch or <2/3 validators online | Check key insertion, check peer connectivity |
| High memory (>4 GB) | Possible leak or archive mode on limited RAM | Restart node, check `--pruning` setting |
