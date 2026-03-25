---
description: How to join the Materios network as a validator or attestor
---

# Operator Guide

There are **two ways** to participate in the Materios network and earn tMATRA rewards:

| Role | What You Do | Rewards | Approval |
|------|------------|---------|----------|
| **Full Validator** | Produce blocks + finalize + attest | Block rewards + attestation rewards | Invite required |
| **Attestor** | Verify blobs and sign attestations | Attestation rewards | No approval needed |

Both roles contribute to network security. Full validators secure consensus (block production + finality). Attestors secure data integrity (verifying that game scores are real).

***

## Attestor (Permissionless)

Anyone can become an attestor. No invite token, no approval, no waiting. You run a single command and start earning tMATRA for every receipt you help certify.

### Requirements

| Requirement | Details |
|-------------|---------|
| **Docker** | Docker Engine 20+ with Compose v2 |
| **CPU** | 1 vCPU |
| **RAM** | 512 MB |
| **Disk** | 1 GB |
| **Network** | Outbound HTTPS + WSS only (no ports to open) |
| **OS** | Linux, macOS, or Windows with Docker |

### Quick Start

```bash
curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install.sh | bash -s -- --mode attestor
```

That's it. The installer:

1. Checks Docker and system requirements
2. Generates a fresh sr25519 keypair
3. Pulls the cert daemon Docker image
4. Writes a `docker-compose.yml` with the cert daemon connected to the public Materios RPC
5. Starts the cert daemon
6. Submits a `join_committee` transaction to add you to the attestor set on-chain
7. Begins polling for receipts to verify

When it finishes, you'll see:

```
  Materios Attestor Online

  SS58 Address   : 5YourAddress...
  Label          : your-node-name
  Mode           : attestor (cert daemon only)
  Daemon Health  : http://localhost:8080/status
  Mnemonic       : ~/materios-attestor/.secret-mnemonic
```

### What Your Daemon Does

1. **Polls** the Materios chain for new unverified receipts
2. **Fetches** the associated blob data from the Materios Blob Gateway
3. **Verifies** blob integrity (SHA-256 chunk hashes match the on-chain content hash)
4. **Validates** blob content against the registered schema (field types, bounds, computed checks)
5. **Signs** an attestation and submits it on-chain
6. **Earns tMATRA** — 10 tMATRA paid instantly each time a receipt you attested reaches certification threshold
7. **Sends heartbeats** every 30 seconds so the network knows you're online

All automatic. No manual intervention.

### Managing Your Attestor

```bash
cd ~/materios-attestor

# View logs
docker compose logs -f cert-daemon

# Check status
curl -s http://localhost:8080/status | python3 -m json.tool

# Restart
docker compose restart

# Stop
docker compose down

# Update to latest version
docker compose pull && docker compose up -d
```

***

## Full Validator (Invite Required)

Full validators run a Materios node (block production + finality) **and** a cert daemon (attestation). You earn from both reward pools.

### Requirements

| Requirement | Details |
|-------------|---------|
| **Docker** | Docker Engine 20+ with Compose v2 |
| **CPU** | 2+ vCPU |
| **RAM** | 2 GB minimum |
| **Disk** | 50 GB SSD |
| **Network** | Port 30333 open inbound (P2P), outbound HTTPS + WSS |
| **OS** | Linux (x86\_64 or arm64), macOS (Apple Silicon or Intel) |

### Why do validators need an invite?

On testnet, the invite token acts as a stand-in for economic stake. Full validators have consensus power (block production + finality voting), so an unchecked authority set is vulnerable to Sybil attacks at zero cost. On mainnet, the invite is replaced by MATRA staking — anyone who stakes sufficient MATRA can become a validator without approval.

### Quick Start

#### 1. Get an invite token

Join the [Flux Point Studios Discord](https://discord.gg/MfYUMnfrJM) and request an invite token in the #materios channel. You will receive a single-use token (64-character hex string).

#### 2. Open port 30333

Your node needs port **30333 TCP** open for incoming P2P connections.

```bash
# UFW example
sudo ufw allow 30333/tcp
```

#### 3. Run the installer

```bash
curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install.sh | bash -s -- --token YOUR_INVITE_TOKEN
```

The installer handles everything:

* Checks that Docker, Compose v2, disk, and RAM meet requirements
* Pulls the validator node and cert daemon Docker images
* Generates a fresh sr25519 keypair
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

#### 4. Back up your mnemonic

Your 24-word mnemonic is saved at `~/materios-operator/.secret-mnemonic`. **Back it up immediately.** This is your validator identity.

#### 5. Wait for activation

After your node syncs and reports session keys, the Materios team will:

1. Add you to the validator authority set (Aura block production + Grandpa finality)
2. Fund your account with MATRA tokens

Your cert daemon automatically joins the attestation committee on-chain (no approval needed for attestation — only block production requires activation).

### What Your Validator Does

#### Validator Node (materios-node)

1. **Connects** to the Materios P2P network via port 30333
2. **Syncs** the full blockchain state
3. **Produces blocks** when it's your turn in the Aura round-robin schedule
4. **Participates in Grandpa finality** — votes to finalize blocks with BFT consensus
5. **Serves RPC** locally for the cert daemon

#### Cert Daemon

Same as the attestor role above — polls, fetches, verifies, attests, earns.

### Managing Your Validator

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

## Rewards

Operators earn tMATRA from two separate reward pools:

### Block Production Rewards (Full Validators Only)

| Parameter | Value |
|-----------|-------|
| **Reserve** | 150M MATRA (15% of total supply) |
| **Era length** | ~24 hours (14,400 blocks) |
| **Distribution** | Proportional to blocks produced per era |
| **Mechanism** | Automatic — credited at each era boundary |

Validators who produce more blocks (better uptime) earn a larger share. Offline during an era = missed rewards (no slashing).

### Attestation Rewards (All Attestors)

| Parameter | Value |
|-----------|-------|
| **Reserve** | 50M MATRA (5% of total supply) |
| **Reward per certification** | 10 tMATRA per signer |
| **Distribution** | Instant — paid the moment a receipt reaches threshold |
| **Mechanism** | Automatic — every attestor who signed receives the reward |

Attestation rewards are earned by **both** full validators and standalone attestors. Every time you help certify a receipt (by signing an attestation that meets the committee threshold), you receive 10 tMATRA immediately.

### Reward Comparison

| | Full Validator | Attestor Only |
|---|---|---|
| **Block rewards** | ~14.7K tMATRA/day (split among validators) | - |
| **Attestation rewards** | 10 tMATRA per certified receipt | 10 tMATRA per certified receipt |
| **Hardware cost** | 2 vCPU, 2 GB RAM, 50 GB SSD | 1 vCPU, 512 MB RAM, 1 GB |
| **Network** | Port 30333 open | Outbound only |
| **Approval** | Invite token (testnet only — replaced by staking on mainnet) | None |

***

## Optional: Custom Label

```bash
# Attestor
curl -sSL .../install.sh | bash -s -- --mode attestor --label "my-attestor-01"

# Full validator
curl -sSL .../install.sh | bash -s -- --token YOUR_TOKEN --label "my-datacenter-01"
```

***

## Health Endpoints

The cert daemon exposes a local HTTP server on port 8080:

| Endpoint | Purpose |
|----------|---------|
| `GET /health` | Basic liveness check. Returns `200` if the process is running. |
| `GET /ready` | Readiness check. Returns `200` if connected to the chain and recently polled. |
| `GET /status` | Full daemon state: block height, finality gap, pending receipts, uptime, version. |
| `GET /metrics` | Prometheus metrics for monitoring integration. |

***

## If Sync Times Out (Full Validators)

If your node is still syncing when the installer finishes, session keys won't be generated automatically. Once the node catches up:

```bash
cd ~/materios-operator
bash generate-session-keys.sh
```

***

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Installer fails at "Pulling image" | Docker not running or no internet | Start Docker: `sudo systemctl start docker` |
| "Invalid invite token" | Token already used or mistyped | Request a new token (full validators only) |
| Node stuck syncing | No peers found | Ensure port 30333 is open: `sudo ufw allow 30333/tcp` |
| `/ready` returns 503 | Daemon still connecting | Wait 1-2 minutes. Check logs: `docker compose logs -f cert-daemon` |
| Heartbeat not showing | Daemon not yet connected | Check daemon logs for RPC connection |
| "Permission denied" on Docker | User not in docker group | Run: `sudo usermod -aG docker $USER` then log out/in |
| "AlreadyCommitteeMember" | Daemon tried to join but already in committee | Normal — the daemon handles this gracefully |
| Not earning attestation rewards | Account not funded with MATRA for TX fees | Request testnet tMATRA in Discord |

***

## Security

* **Your mnemonic never leaves your machine.** Generated locally inside a throwaway container.
* **Session keys are generated locally.** Only public keys are reported to the gateway.
* **RPC is localhost-only.** Port 9944 is bound to 127.0.0.1 (full validators).
* **P2P port 30333 only carries Substrate protocol traffic** (full validators).
* **Heartbeat signatures are verifiable.** Signed with sr25519 — anyone can verify on-chain.
* **Attestors use the public RPC.** No sensitive ports are opened.

***

## Architecture

### Full Validator

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

### Attestor Only

```
┌────────────────────────────────┐
│       Your Machine              │
│                                 │
│  ┌──────────────┐               │
│  │ cert-daemon  │               │
│  │              │               │
│  │ Attestation  │               │
│  │ Heartbeats   │               │
│  └──────┬───────┘               │
│         │                       │
│    port 8080                    │
│    (health, local)              │
└─────────┼───────────────────────┘
          │
          ▼
    Public Materios RPC    Blob Gateway
    (wss://materios...)    (HTTPS)
```
