---
description: End-to-end guide to running a Materios preprod validator (SPO or permissioned, you do not need to be a Cardano SPO) or a permissionless attestor.
---

# Operator Guide

Three ways to participate in Materios preprod and earn tMATRA:

| Role | What you do | Rewards | Requires |
|------|-------------|---------|----------|
| **SPO Validator** | Produce blocks + vote on finality + attest. Selected via Ariadne weighted by Cardano preprod-ada stake (2 of 5 committee seats). | Block rewards + attestation rewards. | A registered Cardano preprod stake pool with delegated stake. ~10 days for stake-snapshot to settle. |
| **Permissioned Validator** | Same chain duties as SPO Validator. Selected from the operator-managed permissioned-candidate list (3 of 5 committee seats). | Block rewards + attestation rewards. | Send your Materios pubkeys to Flux Point Studios. **No Cardano stake pool, no tADA, no 10-day wait.** Hardware spec is identical to SPO Validator. |
| **Attestor** | Verify blobs and sign attestations. | Attestation rewards only. | Nothing. One-line install. |

All three contribute to network security. Validators secure consensus; attestors secure data integrity (confirming that each receipt corresponds to real blob content).

> **Choosing between SPO and Permissioned Validator:** If you already run a Cardano stake pool and want a second revenue stream that uses your existing infrastructure, go SPO. If you don't have a Cardano pool but you do operate cloud / home infrastructure and want to validate on Materios directly, go Permissioned. The hardware spec, chain duties, and reward rate are identical — only the path onto the candidate list differs.

Read [Node Requirements](node-requirements.md) first — it enumerates the downloads, ports, and hardware referenced below.

---

## SPO Validator

Registration happens on Cardano preprod, not on Materios. Once you're registered and your stake snapshot has settled, Ariadne will select your pool for the 2 open committee seats with probability proportional to your active preprod-ada stake.

**Total time commitment:**
- Setup + registration: ~30 minutes of hands-on work.
- **Stake-snapshot wait: 2 full preprod epochs (~10 days).** This is a Cardano protocol delay, not something we control.
- First committee selection: the epoch after your registration's snapshot becomes "go".

### Prerequisites

Already running a Cardano preprod stake pool? Skip to step 1.

If not, follow the official Cardano SPO docs — [relay node configuration](https://developers.cardano.org/docs/operate-a-stake-pool/relay-configuration/relay-node-configuration/) and [block producer wallet/key generation](https://developers.cardano.org/docs/operate-a-stake-pool/block-producer/generating-wallet-keys/) — to produce:
- Pool cold (`cold.skey`, `cold.vkey`, `cold.counter`), VRF (`vrf.skey`, `vrf.vkey`), KES (`kes.skey`, `kes.vkey`), op-cert (`op.cert`).
- Registered stake pool with pledge met.
- At least one payment address funded with a few hundred tADA you control (for the registration tx + a reserved UTXO).

You also need, from [Node Requirements](node-requirements.md):
- `cardano-db-sync` Postgres for preprod (yours or a managed provider — [TxPipe Dolos](https://txpipe.io/) or [Demeter.run](https://demeter.run/) are the usual answers if you don't want to host it).
- Ogmios endpoint pointed at the same cardano-node preprod.
- A box that meets the validator hardware spec.

Set a few env vars to avoid repeating yourself:

```bash
export GENESIS_UTXO="0bacdb7e50ba61a1f9e28007a4f9543fa0e8e31ce10027b2f1dda8ab3438d388#0"
export OGMIOS_URL="ws://<your-ogmios-host>:1337"
export PAYMENT_SKEY=/path/to/your/pool/payment.skey
export COLD_SKEY=/path/to/your/pool/cold.skey
```

`GENESIS_UTXO` is a Materios preprod constant; the others are yours.

### 1. Download the IOG Partner Chains CLI

```bash
curl -sSLo partner-chains-node \
  https://github.com/input-output-hk/partner-chains/releases/download/v1.8.0/partner-chains-node-v1.8.0-x86_64-linux
chmod +x partner-chains-node
sudo mv partner-chains-node /usr/local/bin/
partner-chains-node --version   # → 1.8.0-...
```

This is a standalone tool for registering with any partner chain. It doesn't run a node; it just signs transactions and queries Cardano / Materios state.

### 2. Generate Materios keys

```bash
mkdir -p ~/materios-validator/keys
cd ~/materios-validator/keys

# Sidechain (ECDSA — partner-chain identity)
partner-chains-node key generate --scheme ecdsa > sidechain.json
# Aura (sr25519 — block authoring)
partner-chains-node key generate --scheme sr25519 > aura.json
# Grandpa (ed25519 — finality)
partner-chains-node key generate --scheme ed25519 > grandpa.json

chmod 600 *.json
```

Each `.json` contains mnemonic, public key (hex + SS58), and secret seed. Back them up offline — losing the sidechain key means re-registering from scratch.

Extract the public hexes; you'll need them twice:

```bash
SIDECHAIN_PUB=$(jq -r '.publicKey' sidechain.json)   # 66-char hex, starts 02... or 03...
AURA_PUB=$(jq -r '.publicKey' aura.json)             # 64-char hex
GRANDPA_PUB=$(jq -r '.publicKey' grandpa.json)       # 64-char hex
```

### 3. Pick a registration UTXO

`smart-contracts register` consumes a single UTXO from your pool payment address as a uniqueness nonce.

```bash
cardano-cli query utxo \
  --testnet-magic 1 \
  --address $(cat /path/to/pool/payment.addr)

# Pick any UTXO with ≥ 3 tADA:
export REGISTRATION_UTXO="<txhash>#<ix>"
```

If your address only has one big UTXO, split off a small one first (`cardano-cli transaction build` is fine for this).

### 4. Generate registration signatures

```bash
SIDECHAIN_SIGNING_KEY=$(jq -r '.secretSeed' ~/materios-validator/keys/sidechain.json)
COLD_SIGNING_KEY=$(jq -r '.cborHex' "$COLD_SKEY" | cut -c5-)   # strips 5820 CBOR prefix

partner-chains-node registration-signatures \
  --genesis-utxo "$GENESIS_UTXO" \
  --registration-utxo "$REGISTRATION_UTXO" \
  --mainchain-signing-key "$COLD_SIGNING_KEY" \
  --sidechain-signing-key "$SIDECHAIN_SIGNING_KEY" \
  > phase-a.json

cat phase-a.json
```

Output contains four hex fields — copy them:

```
{
  "spo_public_key": "0x...",
  "spo_signature": "0x...",
  "sidechain_public_key": "0x...",
  "sidechain_signature": "0x..."
}
```

This step is fully offline. The signatures are bound to the `GENESIS_UTXO` + `REGISTRATION_UTXO` pair so they can't be replayed against other partner chains or re-registrations.

### 5. Submit the registration

```bash
SPO_PUB=$(jq -r '.spo_public_key' phase-a.json)
SPO_SIG=$(jq -r '.spo_signature' phase-a.json)
SIDE_SIG=$(jq -r '.sidechain_signature' phase-a.json)

partner-chains-node smart-contracts register \
  --genesis-utxo "$GENESIS_UTXO" \
  --registration-utxo "$REGISTRATION_UTXO" \
  --ogmios-url "$OGMIOS_URL" \
  --payment-key-file "$PAYMENT_SKEY" \
  --partner-chain-public-keys "${SIDECHAIN_PUB}:${AURA_PUB}:${GRANDPA_PUB}" \
  --spo-public-key "$SPO_PUB" \
  --spo-signature "$SPO_SIG" \
  --partner-chain-signature "$SIDE_SIG"
```

Successful output ends with `Transaction submitted. ID: <txhash>`. Note the hash — you'll use it to confirm the registration made it on-chain.

Verify on Cardano preprod:

```bash
partner-chains-node registration-status \
  --chain /path/to/chain-spec-v5-raw.json \
  --mainchain-pub-key "$SPO_PUB" \
  --mainchain-epoch <current-preprod-epoch>
```

Epoch-lookup tip: [preprod.cexplorer.io](https://preprod.cexplorer.io/) shows the current epoch on the front page.

### 6. Wait for the stake snapshot

Ariadne reads from the **2-epoch-stable** Cardano stake snapshot, so your registration becomes selectable **2 preprod epochs (≈ 10 days) after the epoch it was included in**.

| Epoch X | Epoch X+1 | Epoch X+2 |
|---|---|---|
| Registration tx included. | Snapshot captured ("mark"). | Snapshot becomes "set" → Ariadne uses it from this epoch. |

You can watch the queue with:

```bash
partner-chains-node ariadne-parameters \
  --chain /path/to/chain-spec-v5-raw.json \
  --mainchain-epoch <X+2>
```

Your pool ID appears in `registered_candidates` once you're eligible.

### 7. Set up the node

Two paths: native binary + systemd (recommended for SPO operators already running systemd-managed cardano-node), or Docker container (simpler if you prefer containers). Pick one and follow it through step 8.

#### Path A — native binary

Download the binary, WASM override, and chain spec:

```bash
cd /opt
sudo mkdir -p materios/{data,runtime-overrides,keystore}

# Binary
sudo curl -fsSLo /usr/local/bin/materios-node \
  https://materios.fluxpointstudios.com/releases/materios-node-v5-x86_64-linux
sudo chmod +x /usr/local/bin/materios-node

# Runtime override (Ariadne dedup + IDP-None fallback)
sudo curl -fsSLo /opt/materios/runtime-overrides/materios_runtime.compact.compressed.wasm \
  https://materios.fluxpointstudios.com/releases/materios_runtime.compact.compressed.wasm

# Chain spec
sudo curl -fsSLo /opt/materios/chain-spec-v5-raw.json \
  https://materios.fluxpointstudios.com/chain-spec-v5-raw.json

# Verify
curl -sS https://materios.fluxpointstudios.com/releases/SHA256SUMS
cd /opt/materios && \
  sha256sum /usr/local/bin/materios-node \
            runtime-overrides/materios_runtime.compact.compressed.wasm
# Both hashes must match the published SHA256SUMS.
```

Env file for the Cardano mainchain-follower (`/opt/materios/cardano-env`). Every `MC__` / `CARDANO_` / contract-address value is a preprod constant and must match verbatim — getting a policy ID wrong produces a silent Default (all-zeros) substitution that only surfaces as a block-production panic:

```bash
# === Mainchain follower ===
USE_MAIN_CHAIN_FOLLOWER_MOCK=false

# Your cardano-db-sync Postgres (yours, or a TxPipe/Demeter hosted endpoint)
DB_SYNC_POSTGRES_CONNECTION_STRING=postgresql://postgres:PASSWORD@HOST:PORT/cexplorer

# === Cardano preprod epoch config (DO NOT CHANGE) ===
MC__FIRST_EPOCH_TIMESTAMP_MILLIS=1655683200000
MC__EPOCH_DURATION_MILLIS=432000000
MC__FIRST_EPOCH_NUMBER=4
MC__FIRST_SLOT_NUMBER=0

# === Cardano preprod protocol constants (DO NOT CHANGE) ===
CARDANO_SECURITY_PARAMETER=432
CARDANO_ACTIVE_SLOTS_COEFF=0.05
BLOCK_STABILITY_MARGIN=0

# === Materios partner-chain identity (DO NOT CHANGE) ===
PARTNER_CHAIN_GENESIS_UTXO=0bacdb7e50ba61a1f9e28007a4f9543fa0e8e31ce10027b2f1dda8ab3438d388#0

# === Deployed Cardano preprod contract addresses (DO NOT CHANGE) ===
COMMITTEE_CANDIDATE_ADDRESS=addr_test1wzr6en3y43437qps5wscegufxw0euspmy0c3976mjm95j0cwuvezm
D_PARAMETER_POLICY_ID=0x7f57bb675447c65ba0d68270a6b9b93aecc8dfdacaa3aa8cd081f9f3
PERMISSIONED_CANDIDATES_POLICY_ID=0x70cd1c6fbbbd7b1e855f589abd842f433ec0d7b46c7a9e437194e931

# === Your beneficiary — where your block rewards land ===
# This is the 32-byte public key (64 hex chars, NO 0x prefix) of the Materios
# account you want block-production rewards credited to. Use the aura pubkey
# you generated in step 2, OR any Materios sr25519 account you control.
SIDECHAIN_BLOCK_BENEFICIARY=<your-64-char-hex-pubkey>
```

Source it from your systemd unit via `EnvironmentFile=/opt/materios/cardano-env`.

Create an operator system user:

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin materios
sudo chown -R materios:materios /opt/materios
```

#### Path B — Docker

Image: `ghcr.io/flux-point-studios/materios-node:v5` (`:spec-201` and `:latest` also point here). Publicly pullable from GHCR.

`/opt/materios/docker-compose.yml`:

```yaml
services:
  materios-node:
    image: ghcr.io/flux-point-studios/materios-node:v5
    container_name: materios-validator
    restart: unless-stopped
    user: "1000:1000"
    env_file: /opt/materios/cardano-env
    ports:
      - "30333:30333"
      - "127.0.0.1:9944:9944"
    volumes:
      - ./data:/data
      - ./chain-spec-v5-raw.json:/chain-spec/chain-spec-v5-raw.json:ro
      - ./runtime-overrides:/runtime-overrides:ro
      - ./keystore:/keystore
    command:
      - --chain=/chain-spec/chain-spec-v5-raw.json
      - --base-path=/data
      - --keystore-path=/keystore
      - --validator
      - --name=<your-pool-ticker>
      - --port=30333
      - --rpc-port=9944
      - --public-addr=/ip4/<YOUR.PUBLIC.IP>/tcp/30333
      - --wasm-runtime-overrides=/runtime-overrides
      - --bootnodes=/ip4/166.70.250.197/tcp/30333/p2p/12D3KooWPueKoxRAirTTKH4Y2qQAsJDegWMjS4k89Z7izCbZKgkM
```

Prep + start:

```bash
cd /opt/materios
mkdir -p data runtime-overrides keystore
sudo chown -R 1000:1000 data runtime-overrides keystore
curl -fsSLo chain-spec-v5-raw.json https://materios.fluxpointstudios.com/chain-spec-v5-raw.json
curl -fsSLo runtime-overrides/materios_runtime.compact.compressed.wasm \
  https://materios.fluxpointstudios.com/releases/materios_runtime.compact.compressed.wasm
docker compose up -d
docker compose logs -f materios-node
```

Skip to step 9 for key insertion. The Docker path doesn't need the systemd unit below.

### 8. systemd unit (Path A only)

`/etc/systemd/system/materios-validator.service`:

```ini
[Unit]
Description=Materios Preprod Validator
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=materios
Group=materios
EnvironmentFile=/opt/materios/cardano-env
WorkingDirectory=/opt/materios
ExecStart=/usr/local/bin/materios-node \
  --chain /opt/materios/chain-spec-v5-raw.json \
  --base-path /opt/materios/data \
  --keystore-path /opt/materios/keystore \
  --validator \
  --name "<your-pool-ticker>" \
  --port 30333 \
  --rpc-port 9944 \
  --wasm-runtime-overrides /opt/materios/runtime-overrides \
  --bootnodes /ip4/166.70.250.197/tcp/30333/p2p/12D3KooWPueKoxRAirTTKH4Y2qQAsJDegWMjS4k89Z7izCbZKgkM
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

Start it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now materios-validator
journalctl -u materios-validator -f
```

Expect `💤 Idle (N peers), best: #X (…), finalized #Y (…)` within a minute. Verify sync finished:

```bash
curl -s -X POST http://127.0.0.1:9944 \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"system_health"}'
```

`isSyncing: false` before proceeding.

### 9. Insert keys

Use the mnemonics you saved in step 2. Keys are inserted via the local RPC:

```bash
SIDE_MNEMONIC=$(jq -r '.secretPhrase' ~/materios-validator/keys/sidechain.json)
AURA_MNEMONIC=$(jq -r '.secretPhrase' ~/materios-validator/keys/aura.json)
GRANDPA_MNEMONIC=$(jq -r '.secretPhrase' ~/materios-validator/keys/grandpa.json)

for KEY in aura gran crch; do
  case $KEY in
    aura) M="$AURA_MNEMONIC"; P="$AURA_PUB" ;;
    gran) M="$GRANDPA_MNEMONIC"; P="$GRANDPA_PUB" ;;
    crch) M="$SIDE_MNEMONIC"; P="$SIDECHAIN_PUB" ;;
  esac
  curl -s -X POST http://127.0.0.1:9944 \
    -H 'content-type: application/json' \
    -d "{\"jsonrpc\":\"2.0\",\"id\":1,\"method\":\"author_insertKey\",\"params\":[\"$KEY\",\"$M\",\"$P\"]}"
  echo
done
```

Key types: `aura`, `gran` (Grandpa), `crch` (cross-chain / sidechain ECDSA).

Restart the node once to pick up the Grandpa key:

```bash
sudo systemctl restart materios-validator
```

### 10. Confirm eligibility

Once your stake snapshot has settled (step 6) and your node is running, verify you're in the eligible candidate pool:

```bash
partner-chains-node ariadne-parameters \
  --chain /opt/materios/chain-spec-v5-raw.json \
  --mainchain-epoch <current-preprod-epoch>
```

Your `sidechain_pub_key` should appear in `registered_candidates` with a `stake_pool_pub_key` matching your cold key.

After the next Materios session boundary, check the committee:

```bash
# Via any Materios RPC (yours or public)
curl -s -X POST http://127.0.0.1:9944 \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"state_getStorage","params":["<currentCommittee storage key>"]}'
```

Easier: open the [explorer](https://fluxpointstudios.com/materios/explorer) committee tab — your sidechain pubkey shows up by its SS58 derivation.

### Operating

| Task | Command |
|---|---|
| Logs | `journalctl -u materios-validator -f` |
| Restart | `sudo systemctl restart materios-validator` |
| Peer count | `curl -s -X POST http://127.0.0.1:9944 -H 'content-type: application/json' -d '{"jsonrpc":"2.0","id":1,"method":"system_health"}'` |
| Finalized head | `curl -s -X POST http://127.0.0.1:9944 -H 'content-type: application/json' -d '{"jsonrpc":"2.0","id":1,"method":"chain_getFinalizedHead"}'` |
| Upgrade binary / WASM | Re-download → verify sha256 → restart unit |

### KES renewal

Cardano KES op-certs expire ~every 90 days on mainnet / ~9 days on preprod. Re-issue the op-cert with a fresh KES period and restart your cardano-node. **No Materios action is needed** — your partner-chain registration is independent of KES. As long as your Cardano pool is minting blocks, your delegation stays active.

### Rotating Materios keys

If you ever need to rotate your Materios keys (compromised host, etc.), submit a new `smart-contracts register` with the new keys. The old registration remains until you consume a UTXO to deregister (`smart-contracts deregister`). Wait 2 epochs for the new snapshot before expecting selection with the rotated keys.

### What happens if you go offline?

- **Selected for the committee but offline:** the slots you'd have minted go unclaimed; your Grandpa vote is simply absent. No slashing on Materios preprod. Finality stays healthy as long as 2f+1 of the committee stays online (that's 2-of-3 for 3 permissioned + 2 registered at the current D parameter).
- **Missed a whole epoch:** zero block rewards for that epoch. No penalty on future selection.
- **Pool depledged / Cardano-side issues:** your partner-chain registration stays on-chain but you drop out of Ariadne's weighted pool once the Cardano snapshot stops including you. Re-register normally once your Cardano pool is healthy.

---

## Permissioned Validator (non-SPO)

You don't need a Cardano stake pool to validate Materios. The chain has 3 permissioned-candidate seats per committee (D-parameter `(3, 2)`); these are filled from a governance-managed allowlist. **Apply to be added** and you'll be eligible at the next session boundary (~6 minutes), not 2 Cardano epochs.

This is the right path for operators who:
- Already run cloud / home-lab infrastructure but don't run a Cardano stake pool.
- Want to validate Materios as a primary activity (not as a sidecar to existing SPO ops).
- Prefer to avoid the Cardano-side complexity (pool pledge, KES rotation, db-sync hosting if you don't already need it).

The chain duties, reward rate, hardware spec, and operational expectations are **identical** to the SPO Validator path. The only difference is how you get on the candidate list.

### 1. Download the IOG Partner Chains CLI

Same as for SPOs — you only use it for key generation, no Cardano-side commands needed.

```bash
curl -sSLo partner-chains-node \
  https://github.com/input-output-hk/partner-chains/releases/download/v1.8.0/partner-chains-node-v1.8.0-x86_64-linux
chmod +x partner-chains-node
sudo mv partner-chains-node /usr/local/bin/
partner-chains-node --version   # → 1.8.0-...
```

### 2. Generate Materios keys

Same three keys as the SPO path:

```bash
mkdir -p ~/materios-validator/keys
cd ~/materios-validator/keys

# Sidechain (ECDSA — partner-chain identity)
partner-chains-node key generate --scheme ecdsa > sidechain.json
# Aura (sr25519 — block authoring)
partner-chains-node key generate --scheme sr25519 > aura.json
# Grandpa (ed25519 — finality)
partner-chains-node key generate --scheme ed25519 > grandpa.json

chmod 600 *.json
```

Each `.json` contains mnemonic, public key (hex + SS58), and secret seed. **Back them up offline** — losing the sidechain key means re-applying for a permissioned slot.

Extract the public hexes — you'll send these (only the public parts) to Flux Point Studios:

```bash
SIDECHAIN_PUB=$(jq -r '.publicKey' sidechain.json)   # 66-char hex (33-byte compressed ECDSA)
AURA_PUB=$(jq -r '.publicKey' aura.json)             # 64-char hex
GRANDPA_PUB=$(jq -r '.publicKey' grandpa.json)       # 64-char hex
```

### 3. Apply for a permissioned slot

Send the three pubkeys + a label / contact to Flux Point Studios. Easiest path: open an issue in [materios](https://github.com/Flux-Point-Studios/materios/issues) titled `Permissioned validator application: <your-label>` with this body:

```
sidechain_pub: 0x<66-char-hex>
aura_pub:      0x<64-char-hex>
grandpa_pub:   0x<64-char-hex>
label:         <your-display-name>
contact:       <discord-handle or email>
```

Or DM Flux Point Studios in the operator Discord with the same fields.

We will:
1. Confirm the keys decode cleanly.
2. Submit a partner-chains governance tx adding your sidechain pubkey to the permissioned-candidates list on Cardano preprod.
3. Reply with the Cardano tx hash and the Materios session in which you'll first be eligible.

Stake pool ownership is **not checked**. The candidates list is governed by the partner-chains pallet, separate from Cardano stake pool registration.

### 4. Set up the node

Same steps as the SPO path — see [SPO Validator → Set up the node](#7-set-up-the-node). **Skip the Ogmios setup unless you're also planning to register as an SPO later.** The mainchain follower still needs `cardano-db-sync` Postgres for L1 reads — see [Node Requirements → cardano-db-sync](node-requirements.md#separate-requirement-cardano-db-sync) for hosted-provider options.

### 5. Insert keys

Same as SPO step 9 — `author_insertKey` for `aura`, `gran`, `crch`. After insertion, restart the node once to pick up the Grandpa key.

### 6. Confirm eligibility

Once Flux Point Studios confirms your candidates-list addition has landed on Cardano preprod, your sidechain pubkey will appear in the **permissioned** track of `ariadne-parameters`:

```bash
partner-chains-node ariadne-parameters \
  --chain /opt/materios/chain-spec-v5-raw.json \
  --mainchain-epoch <current-preprod-epoch>
```

Look for your `sidechain_pub_key` in `permissioned_candidates` (not `registered_candidates` — that's the SPO track).

After the next Materios session boundary (~6 minutes from your tx landing), check the [explorer](https://fluxpointstudios.com/materios/explorer) committee tab — your SS58 should appear when selected.

### Operating

Identical to the SPO Validator path. See [SPO Validator → Operating](#operating).

### What if you also want to register as an SPO later?

You can. Run the steps in [SPO Validator](#spo-validator) using the **same** sidechain/aura/grandpa keys. Your registration becomes eligible after the 2-epoch stake snapshot, and you'll be selectable from **either** the permissioned track or the SPO track. (Selection is mutually exclusive per session — you'll fill at most one seat per session.)

### Removing yourself from the permissioned list

Open an issue or DM us. We submit a partner-chains governance tx to remove your sidechain pubkey. Takes effect at the next Cardano stake snapshot (~5 days). You can also just stop running the node — your seat falls back to other candidates without any tx.

---

## Attestor (Permissionless)

Anyone can run an attestor. One install command; no invite, no keys to ship, no approval step. You earn 10 tMATRA every time a receipt you signed reaches the certification threshold.

### Requirements

| Resource | Value |
|---|---|
| Docker | Engine 20+ with Compose v2 |
| CPU | 1 vCPU |
| RAM | 512 MB |
| Disk | 1 GB |
| Network | Outbound HTTPS + WSS only |
| OS | Linux, macOS, or Windows (WSL2 + Docker Desktop) |

### Install

```bash
curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install.sh \
  | bash -s -- --mode attestor --label "my-attestor-01"
```

Or download a double-clickable installer:

| Platform | Download |
|----------|----------|
| macOS | [install-macos.command](https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install-macos.command) |
| Windows | [install-windows.bat](https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install-windows.bat) |
| Linux | [install-linux.sh](https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install-linux.sh) |

The installer:

1. Checks Docker / system requirements.
2. Generates a fresh sr25519 attestor keypair.
3. Pulls the cert-daemon image.
4. Writes `~/materios-attestor/docker-compose.yml` pointing at the public Materios RPC (`wss://materios.fluxpointstudios.com/preprod-rpc`).
5. Registers your SS58 with the gateway; the gateway auto-drips MATRA.
6. Starts the daemon; it auto-joins the attestation committee.

On success:

```
  Materios Attestor Online

  SS58 Address   : 5Your...
  Label          : my-attestor-01
  Mode           : attestor (cert daemon only)
  Health         : http://localhost:8080/status
  Mnemonic       : ~/materios-attestor/.secret-mnemonic
```

**Back up `~/materios-attestor/.secret-mnemonic` immediately.** It's your attestor identity.

### What the daemon does

1. Polls Materios for new unverified receipts.
2. Downloads blobs from the blob gateway.
3. Verifies SHA-256 chunk integrity + schema-level content validity.
4. Signs an attestation and submits on-chain.
5. Sends a heartbeat every 30 s so the gateway knows you're online.

Every time one of your attestations helps a receipt clear the threshold, 10 tMATRA lands in your account.

### Operating

```bash
cd ~/materios-attestor
docker compose logs -f cert-daemon        # follow logs
curl -s http://localhost:8080/status      # JSON status
docker compose restart                    # restart
docker compose down                       # stop

# Update (handles chain forks too)
curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/update.sh | bash
```

### Full Validator from operator-kit

Historically operator-kit shipped a `--mode validator` that provisioned a Substrate node and requested addition to the permissioned-candidate list. That mode is being retired in favour of the manual flow documented above in [Permissioned Validator (non-SPO)](#permissioned-validator-non-spo) — same outcome, more operator control over node placement and key custody. If you're an existing operator-kit `--mode validator` user, your setup keeps working; for new validators, follow the explicit Permissioned Validator path.

---

## Rewards

| Pool | Reserve | Reward | Who earns |
|---|---|---|---|
| Block production | 150M MATRA (15% of total supply) | Per-block minter credit, distributed at era end | All Validators (SPO + Permissioned) |
| Attestation | 50M MATRA (5%) | 10 tMATRA per certification per signer, paid instantly | All signers (Validators + Attestors) |

Era length is ~24 hours (14,400 blocks). Block reward is proportional to blocks minted per era — better uptime = bigger share. No slashing for downtime on preprod. Both validator paths (SPO and Permissioned) earn at the same rate per block produced.

---

## Health Endpoints

Cert-daemon local HTTP (port 8080):

| Path | Purpose |
|---|---|
| `GET /health` | Liveness. 200 if process is up. |
| `GET /ready` | Readiness. 200 if connected to chain + polled recently. |
| `GET /status` | Full state: height, finality gap, pending receipts, uptime, version. |
| `GET /metrics` | Prometheus. |

Node RPC (9944, localhost): standard Substrate RPC surface. `system_health`, `chain_getHeader`, `chain_getFinalizedHead`, `state_getStorage`, `author_insertKey`, etc.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `registration-signatures` errors on `mainchain-signing-key` | Didn't strip the 5820 CBOR prefix | `jq -r '.cborHex' cold.skey \| cut -c5-` |
| `smart-contracts register` fails with `UTxO already spent` | Your `$REGISTRATION_UTXO` was consumed between prep + submit | Pick a fresh UTXO; re-sign (same sidechain / spo keys are fine). |
| Node starts then crashes on first MC epoch boundary | Runtime override missing or wrong hash | Re-download WASM, verify sha256, restart |
| `registration-status` says "not registered" 10 minutes after submit | Cardano preprod hasn't included the tx yet | Wait for 2 blocks + check the txhash on [preprod.cexplorer.io](https://preprod.cexplorer.io/) |
| Registered but `ariadne-parameters` doesn't list you | Stake snapshot hasn't rolled forward yet | You need to be past epoch X+2; see step 6 |
| In `registered_candidates` but never selected | Stake too low relative to other pools | Delegation volume determines selection probability; grow your pool's stake or wait for variance |
| Cert-daemon `State already discarded` loop | Daemon cursor is behind node pruning window | `rm ~/materios-attestor/data/daemon-state.json` + `docker compose restart` |

---

## Security Posture

- Mnemonics are generated locally and never sent anywhere.
- Session keys live only in your node's local keystore.
- Node RPC is bound to 127.0.0.1; **do not** expose `--rpc-external` or `--unsafe-rpc-external` on a public IP.
- P2P port 30333 only carries Substrate protocol traffic.
- Heartbeats are sr25519-signed and verifiable on-chain.
- Cardano signing keys (cold, payment) never leave your SPO host; our tooling doesn't ask for them.

---

## Architecture

### SPO Validator

```
┌────────────────── Your SPO host ──────────────────┐
│  cardano-node (preprod)                           │
│    └─→ cardano-db-sync (Postgres)                 │
│    └─→ Ogmios (WebSocket)                         │
│                                                    │
│  materios-node  ─── P2P :30333 ───→  Materios net │
│    ├─ reads db-sync Postgres                      │
│    ├─ reads Ogmios (for SPO CLI only)             │
│    └─ RPC :9944 (localhost)                       │
│                                                    │
│  cert-daemon  ─── WSS → Materios RPC               │
│    └─ attests + earns 10 tMATRA per cert          │
└────────────────────────────────────────────────────┘
```

### Permissioned Validator

```
┌─────────────── Your validator host ──────────────┐
│  cardano-db-sync (Postgres) — yours OR managed   │
│    (TxPipe / Demeter / Blockfrost — no Cardano   │
│     stake pool needed)                            │
│                                                    │
│  materios-node  ─── P2P :30333 ───→  Materios net │
│    ├─ reads db-sync Postgres                      │
│    └─ RPC :9944 (localhost)                       │
│                                                    │
│  cert-daemon  ─── WSS → Materios RPC               │
│    └─ attests + earns 10 tMATRA per cert          │
└────────────────────────────────────────────────────┘
```

Same chain duties as SPO — the only structural difference is no cardano-node and no Ogmios.

### Attestor only

```
┌── Your machine ──┐
│  cert-daemon     │  ─── WSS ──→ wss://materios.fluxpointstudios.com/preprod-rpc
│  /status :8080   │              └── earns 10 tMATRA per cert
└──────────────────┘
```

---

## Getting help

- Open an issue in [materios](https://github.com/Flux-Point-Studios/materios/issues) for chain / runtime problems.
- Open an issue in [materios-operator-kit](https://github.com/Flux-Point-Studios/materios-operator-kit/issues) for attestor installer / cert-daemon issues.
- Registration stuck on Cardano-side questions → check [preprod.cexplorer.io](https://preprod.cexplorer.io/) for your tx + pool status; the IOG partner-chains docs at [input-output-hk/partner-chains](https://github.com/input-output-hk/partner-chains) cover the lower-level flow we wrap.
