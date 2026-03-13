---
description: How to become a Materios validator in one command
---

# Operator Guide

This guide covers how to join the Materios network as a **validator operator**. Validator operators run a full Materios node (block production + finality) and a cert daemon (threshold attestation) — earning rewards for securing the network.

***

## Prerequisites

| Requirement | Details |
| ----------- | ------- |
| **Docker** | Docker Engine 20+ with Compose v2 |
| **CPU** | 2+ vCPU |
| **RAM** | 2 GB minimum |
| **Disk** | 50 GB SSD |
| **Network** | Port 30333 open inbound (P2P), outbound HTTPS + WSS |
| **OS** | Linux (x86\_64 only) |

***

## Quick Start

### 1. Get an invite token

Join the [Flux Point Studios Discord](https://discord.gg/MfYUMnfrJM) and request an invite token in the #materios channel. You will receive a single-use token that looks like a 64-character hex string.

### 2. Open port 30333

Your node needs port **30333 TCP** open for incoming P2P connections. This is how other validators find and communicate with your node.

```bash
# UFW example
sudo ufw allow 30333/tcp

# iptables example
sudo iptables -A INPUT -p tcp --dport 30333 -j ACCEPT
```

### 3. Run the installer

```bash
curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install.sh | bash -s -- --token YOUR_INVITE_TOKEN
```

That's it. The installer handles everything:

* Checks that Docker, Compose v2, disk, and RAM meet requirements
* Pulls the validator node and cert daemon Docker images
* Generates a fresh sr25519 keypair (inside a container — nothing to install on your host)
* Redeems your invite token to register with the Materios gateway
* Writes a fully configured `docker-compose.yml` with both services
* Starts the validator node and waits for chain sync
* Generates session keys (Aura + Grandpa) for block production and finality
* Reports session keys to the Materios gateway
* Starts the cert daemon for threshold attestation

When it finishes, you'll see:

```
  Materios Validator Online

  SS58 Address   : 5YourAddress...
  Label          : your-node-name
  Session Keys   : 0x1234abcd...ef56
  Peer ID        : 12D3KooW...
  P2P Port       : 30333 (ensure this is open inbound)
  Node RPC       : http://localhost:9944
  Daemon Health  : http://localhost:8080/status
  Explorer       : https://materios.fluxpointstudios.com/explorer/#/committee
  Mnemonic       : ~/materios-operator/.secret-mnemonic
```

### 4. Back up your mnemonic

Your 24-word mnemonic is saved at `~/materios-operator/.secret-mnemonic`. **Back it up immediately** to a password manager or secure offline storage. This is your validator identity — if you lose it, you lose your authority seat.

### 5. Wait for activation

After your node syncs and reports session keys, the Materios team will:

1. Add you to the attestation committee
2. Add you to the validator authority set (Aura block production + Grandpa finality)
3. Fund your account with MATRA tokens

This typically happens within a few hours. You can check your status at any time:

* **Explorer**: [materios.fluxpointstudios.com/explorer/#/committee](https://materios.fluxpointstudios.com/explorer/#/committee) — look for your node's label
* **Local health**: `curl http://localhost:8080/status`
* **Public heartbeat API**: `curl https://materios.fluxpointstudios.com/blobs/heartbeats/status`

***

## What Your Node Does

Once activated, your operator runs two services:

### Validator Node (materios-node)

1. **Connects** to the Materios P2P network via port 30333
2. **Syncs** the full blockchain state
3. **Produces blocks** when it's your turn in the Aura round-robin schedule
4. **Participates in Grandpa finality** — votes to finalize blocks with BFT consensus
5. **Serves RPC** locally for the cert daemon

### Cert Daemon

1. **Polls** for new receipts that need attestation
2. **Fetches** the associated blob data from the gateway
3. **Verifies** blob integrity (SHA-256 chunk hashes)
4. **Signs** an attestation and submits it on-chain
5. **Sends heartbeats** every 30 seconds so the network knows you're online

All of this happens automatically. No manual intervention needed.

***

## Managing Your Node

All commands are run from `~/materios-operator/`:

```bash
cd ~/materios-operator

# View validator node logs
docker compose logs -f materios-node

# View cert daemon logs
docker compose logs -f cert-daemon

# Check node sync status
curl -s -X POST http://localhost:9944 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"system_syncState","params":[]}' | python3 -m json.tool

# Check daemon status (JSON)
curl -s http://localhost:8080/status | python3 -m json.tool

# Restart all services
docker compose restart

# Stop all services
docker compose down

# Update to the latest version
docker compose pull && docker compose up -d
```

***

## Optional: Custom Label

You can pass a `--label` flag during installation to set a human-readable name for your node:

```bash
curl -sSL .../install.sh | bash -s -- --token YOUR_TOKEN --label "my-datacenter-01"
```

If omitted, the installer generates a label from your hostname.

***

## Health Endpoints

The cert daemon exposes a local HTTP server on port 8080:

| Endpoint | Purpose |
| -------- | ------- |
| `GET /health` | Basic liveness check. Returns `200` if the process is running. |
| `GET /ready` | Readiness check. Returns `200` if connected to the chain and recently polled. Returns `503` otherwise. |
| `GET /status` | Full daemon state: block height, finality gap, pending receipts, substrate connection, uptime, version. |
| `GET /metrics` | Prometheus metrics for monitoring integration. |

***

## If Sync Times Out

If your node is still syncing when the installer finishes, session keys won't be generated automatically. Once the node catches up, run:

```bash
cd ~/materios-operator
bash generate-session-keys.sh
```

This generates session keys and reports them to the gateway so the Materios team can activate your authority seat.

***

## Troubleshooting

| Symptom | Likely Cause | Fix |
| ------- | ------------ | --- |
| Installer fails at "Pulling image" | Docker not running or no internet | Start Docker: `sudo systemctl start docker` |
| "Invalid invite token" | Token already used or mistyped | Request a new token from the Materios team |
| "Unsupported architecture" | Not running x86\_64 | The validator node requires amd64 hardware |
| Node stuck syncing | No peers found | Ensure port 30333 is open: `sudo ufw allow 30333/tcp` |
| `/ready` returns 503 | Daemon still connecting | Wait 1–2 minutes. Check logs: `docker compose logs -f cert-daemon` |
| Heartbeat not showing in explorer | Authority seat not yet activated | Contact the Materios team to confirm activation |
| "Permission denied" on Docker | User not in `docker` group | Run: `sudo usermod -aG docker $USER` then log out and back in |
| Blocks not finalizing after activation | Grandpa key mismatch | Check `docker compose logs materios-node` for Grandpa errors |

***

## Security

* **Your mnemonic never leaves your machine.** It is generated locally inside a throwaway container and saved to `~/.secret-mnemonic`.
* **Session keys are generated locally.** The `author_rotateKeys` RPC runs on your local node. Only the public keys are reported to the gateway.
* **The invite token is single-use.** Once redeemed, it cannot be used again.
* **RPC is localhost-only.** Port 9944 is bound to 127.0.0.1 — not accessible from the internet.
* **P2P port 30333 only carries Substrate protocol traffic.** No sensitive data is exposed.
* **Heartbeat signatures are verifiable.** Your daemon signs each heartbeat with sr25519 — anyone can verify your identity on-chain.

***

## Architecture Overview

```
┌─────────────────────────────────────────┐
│          Your Machine                    │
│                                          │
│  ┌──────────────┐   ┌──────────────┐    │
│  │ materios-node│   │ cert-daemon  │    │
│  │              │◄──│              │    │
│  │ Block prod.  │ws │ Attestation  │    │
│  │ Finality     │   │ Heartbeats   │    │
│  └──────┬───────┘   └──────┬───────┘    │
│         │                  │             │
│    port 30333         port 8080          │
│    (P2P inbound)     (health, local)     │
└─────────┼──────────────────┼─────────────┘
          │                  │
          ▼                  ▼
    Materios P2P       Blob Gateway
    Network            (HTTPS)
```

***

## Requirements Summary

| You Need | You Don't Need |
| -------- | -------------- |
| Docker + Compose v2 | Python, Rust, or Node.js |
| 2+ vCPU, 2 GB+ RAM, 50 GB SSD | To build anything from source |
| Port 30333 open inbound | Static IP |
| Outbound HTTPS + WSS | Complex firewall rules |
| An invite token from the Materios team | Prior blockchain experience |
