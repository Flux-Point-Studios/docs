---
description: How to become a Materios attestation operator in one command
---

# Operator Guide

This guide covers how to join the Materios attestation committee as an **external operator**. External operators run a lightweight cert daemon that participates in threshold certification of data blobs — no full Substrate node required.

***

## Prerequisites

| Requirement | Details |
| ----------- | ------- |
| **Docker** | Docker Engine 20+ with Compose v2 |
| **CPU** | 1 vCPU |
| **RAM** | 512 MB |
| **Disk** | 1 GB free |
| **Network** | Outbound HTTPS + WSS (no inbound ports needed) |
| **OS** | Linux (x86\_64 or ARM64) |

***

## Quick Start

### 1. Get an invite token

Contact the [Flux Point Studios](https://materios.fluxpointstudios.com) team to request an invite token. You will receive a single-use token that looks like a 64-character hex string.

### 2. Run the installer

```bash
curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install.sh | bash -s -- --token YOUR_INVITE_TOKEN
```

That's it. The installer handles everything:

* Checks that Docker and Compose v2 are installed
* Pulls the operator Docker image
* Generates a fresh sr25519 keypair (inside a container — nothing to install on your host)
* Redeems your invite token to register with the Materios gateway
* Writes a fully configured `docker-compose.yml`
* Starts the cert daemon
* Waits for the first successful health check

When it finishes, you'll see:

```
  Materios Operator Online

  SS58 Address  : 5YourAddress...
  Label         : your-node-name
  Health        : http://localhost:8080/status
  Explorer      : https://materios.fluxpointstudios.com/explorer/#/committee
  Mnemonic      : ~/materios-operator/.secret-mnemonic
```

### 3. Back up your mnemonic

Your 24-word mnemonic is saved at `~/materios-operator/.secret-mnemonic`. **Back it up immediately** to a password manager or secure offline storage. This is your validator identity — if you lose it, you lose your committee seat.

### 4. Wait for activation

After you register, the Materios team will activate your committee seat on-chain. This typically happens within a few hours. You can check your status at any time:

* **Explorer**: [materios.fluxpointstudios.com/explorer/#/committee](https://materios.fluxpointstudios.com/explorer/#/committee) — look for your node's label
* **Local health**: `curl http://localhost:8080/status`
* **Public heartbeat API**: `curl https://materios.fluxpointstudios.com/blobs/heartbeats/status`

Your daemon will send heartbeats immediately and appear in the explorer as soon as the team activates your seat.

***

## What the Daemon Does

Once running, your cert daemon:

1. **Connects** to the Materios chain via WebSocket (`wss://materios.fluxpointstudios.com/rpc`)
2. **Polls** for new receipts that need attestation
3. **Fetches** the associated blob data from the gateway
4. **Verifies** blob integrity (SHA-256 chunk hashes)
5. **Signs** an attestation and submits it on-chain
6. **Sends heartbeats** every 30 seconds so the network knows you're online

All of this happens automatically. No manual intervention needed.

***

## Managing Your Node

All commands are run from `~/materios-operator/`:

```bash
cd ~/materios-operator

# View live logs
docker compose logs -f

# Check daemon status (JSON)
curl -s http://localhost:8080/status | python3 -m json.tool

# Restart the daemon
docker compose restart

# Stop the daemon
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

The daemon exposes a local HTTP server on port 8080:

| Endpoint | Purpose |
| -------- | ------- |
| `GET /health` | Basic liveness check. Returns `200` if the process is running. |
| `GET /ready` | Readiness check. Returns `200` if connected to the chain and recently polled. Returns `503` otherwise. |
| `GET /status` | Full daemon state: block height, finality gap, pending receipts, substrate connection, uptime, version. |
| `GET /metrics` | Prometheus metrics for monitoring integration. |

***

## Troubleshooting

| Symptom | Likely Cause | Fix |
| ------- | ------------ | --- |
| Installer fails at "Pulling image" | Docker not running or no internet | Start Docker: `sudo systemctl start docker` |
| "Invalid invite token" | Token already used or mistyped | Request a new token from the Materios team |
| `/ready` returns 503 | Daemon still connecting to chain | Wait 1–2 minutes. Check logs: `docker compose logs -f` |
| Heartbeat not showing in explorer | Committee seat not yet activated | Contact the Materios team to confirm activation |
| "Permission denied" on Docker | User not in `docker` group | Run: `sudo usermod -aG docker $USER` then log out and back in |

***

## Security

* **Your mnemonic never leaves your machine.** It is generated locally inside a throwaway container and saved to `~/.secret-mnemonic`.
* **The invite token is single-use.** Once redeemed, it cannot be used again.
* **Heartbeat signatures are verifiable.** Your daemon signs each heartbeat with sr25519 — anyone can verify your identity on-chain.
* **No inbound ports required.** The daemon only makes outbound connections (HTTPS to the gateway, WSS to the chain RPC).

***

## Requirements Summary

| You Need | You Don't Need |
| -------- | -------------- |
| Docker + Compose v2 | Python, Rust, or Node.js |
| 1 vCPU, 512 MB RAM | A full Substrate node |
| Outbound HTTPS + WSS | Inbound ports or static IP |
| An invite token from the Materios team | To build anything from source |
