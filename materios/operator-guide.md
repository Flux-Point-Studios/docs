---
description: How to join the Materios network as a validator or attestor
---

# Operator Guide

There are **two ways** to participate in the Materios network and earn tMATRA rewards:

| Role | What You Do | Rewards | Approval |
|------|------------|---------|----------|
| **Full Validator** | Produce blocks + finalize + attest | Block rewards + attestation rewards | No approval needed |
| **Attestor** | Verify blobs and sign attestations | Attestation rewards | No approval needed |

Both roles contribute to network security. Full validators secure consensus (block production + finality). Attestors secure data integrity (verifying that game scores are real).

> **No API key required.** Submitting receipts to Materios is permissionless — anyone with an sr25519 keypair and MATRA for TX fees (available from the faucet on testnet) can submit. The blob gateway also accepts sr25519-signed uploads without API keys.

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

#### Option A: One-liner (terminal)

```bash
curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install.sh | bash -s -- --mode attestor
```

#### Option B: Download & double-click (no terminal needed)

| Platform | Download | Instructions |
|----------|----------|-------------|
| **macOS** | [install-macos.command](https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install-macos.command) | Download, double-click. If blocked: System Settings → Privacy & Security → Open Anyway |
| **Windows** | [install-windows.bat](https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install-windows.bat) | Download, double-click. Requires WSL2 + Docker Desktop |
| **Linux** | [install-linux.sh](https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install-linux.sh) | Download, run `chmod +x install-linux.sh && ./install-linux.sh` |

All wrappers guide you through mode selection (validator vs attestor) and run the full installer automatically.

That's it. The installer:

1. Checks Docker and system requirements
2. Generates a fresh sr25519 keypair
3. Pulls the cert daemon Docker image
4. Writes a `docker-compose.yml` with the cert daemon connected to the public Materios RPC
5. Requests a MATRA faucet drip (funds your account for transaction fees)
6. Starts the cert daemon
7. Daemon auto-joins the attestation committee on-chain
8. Begins polling for receipts to verify and earn tMATRA rewards

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

# Update (handles chain forks automatically)
curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/update.sh | bash
```

***

## Full Validator

Full validators run a Materios node (block production + finality) **and** a cert daemon (attestation). You earn from both reward pools.

### Requirements

Running a full Materios validator means running **two Docker stacks**: the Materios node + cert-daemon (what the installer sets up) AND a Cardano preprod stack (cardano-node + cardano-db-sync + Postgres) which the Materios node reads committee state from.

| Requirement | Materios stack | Cardano stack |
|-------------|---------------|---------------|
| **Docker** | Engine 20+, Compose v2 | same |
| **CPU** | 2+ vCPU | 2+ vCPU |
| **RAM** | 2 GB | 2 GB |
| **Disk** | 50 GB SSD | 50 GB SSD (grows with preprod chain) |
| **Network** | Port 30333 open inbound, outbound HTTPS/WSS | Outbound only |
| **OS** | Linux, macOS, or Windows with Docker | same |

> **Already run a Cardano SPO?** You almost certainly have db-sync + Postgres already. Skip to the installer step — just export your connection string first.
>
> **Don't want to run a Cardano stack?** Run **attestor mode** (above) instead. You still earn attestation rewards and your SS58 shows on the committee page. The full validator path is for operators who are comfortable running Cardano infrastructure.

### Prerequisite: Cardano preprod stack

If you already have one, skip ahead. Otherwise, this reference Docker Compose brings up cardano-node + db-sync + Postgres on preprod:

<details>
<summary>Reference <code>docker-compose.yml</code> (click to expand)</summary>

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
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s

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

</details>

Bring it up: `docker compose up -d`. Initial sync from genesis takes several hours — accelerate it with a [Mithril snapshot](https://mithril.network/doc/manual/getting-started/bootstrap-cardano-node) if you need to.

Once db-sync is making progress (`docker compose logs -f db-sync-preprod`), export its connection string:

```bash
export DB_SYNC_POSTGRES_CONNECTION_STRING='postgres://postgres:change-me@localhost:5433/cexplorer'
```

Your Materios node will fail fast during install if this isn't set.

### Quick Start

> **Not comfortable with the terminal?** Download the installer for your platform:
>
> | Platform | Download |
> |----------|----------|
> | **macOS** | [install-macos.command](https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install-macos.command) — double-click to run |
> | **Windows** | [install-windows.bat](https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install-windows.bat) — double-click to run |
> | **Linux** | [install-linux.sh](https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install-linux.sh) — `chmod +x && ./install-linux.sh` |

### Terminal Install (Linux / macOS)

#### 1. Open port 30333

Your node needs port **30333 TCP** open for incoming P2P connections.

```bash
# UFW example (Linux)
sudo ufw allow 30333/tcp
```

#### 2. Run the installer

```bash
curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install.sh | bash -s -- --label my-validator
```

No invite token or approval needed. The installer auto-registers via the faucet.

### Quick Start (Windows / PowerShell)

#### Prerequisites

1. **Install Docker Desktop**: Download from [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/). Enable WSL2 backend during installation.
2. **Enable WSL2** (if not already): Open PowerShell as Administrator and run:
   ```powershell
   wsl --install
   ```
   Restart if prompted, then open the **Ubuntu** terminal from the Start menu.
3. **Verify Docker in WSL**: In the Ubuntu terminal:
   ```bash
   docker --version
   docker compose version
   ```

#### Run the installer (inside WSL Ubuntu terminal)

```bash
curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install.sh | bash -s -- --label my-validator
```

> **Note for Windows users:** The installer must run inside WSL (Ubuntu terminal), not native PowerShell. Docker Desktop's WSL integration makes Docker available inside WSL automatically. Port 30333 will need to be opened in Windows Firewall (Settings > Firewall > Advanced Settings > Inbound Rules > New Rule > Port > TCP 30333).

### What the installer does

* Checks that Docker, Compose v2, disk, and RAM meet requirements
* Downloads the official Materios chain spec
* Pulls the validator node (v3) and cert daemon Docker images
* Generates a fresh sr25519 keypair
* Auto-registers with the Materios gateway via faucet
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
  Explorer       : https://fluxpointstudios.com/materios/explorer
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

## Updating Your Node

Run a single command to update images, detect chain forks, and restart:

```bash
curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/update.sh | bash
```

This works for both validators and attestors. If the network has forked to a new chain (e.g. after a consensus reset), the update script automatically:

1. Detects the genesis mismatch
2. Downloads the correct chain spec
3. Wipes chain data (your keystore and identity are preserved)
4. Updates your configuration
5. Restarts and re-syncs
6. Regenerates session keys (validators only — share them with the team)

If there's no fork, it simply pulls latest images and restarts.

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
| **Approval** | None (staking on mainnet) | None |

***

## Optional: Custom Label

```bash
# Attestor
curl -sSL .../install.sh | bash -s -- --mode attestor --label "my-attestor-01"

# Full validator
curl -sSL .../install.sh | bash -s -- --label "my-datacenter-01"
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
    Public Materios RPC                  Blob Gateway
    (wss://materios..../preprod-rpc)     (HTTPS)
```

