---
description: End-to-end guide to running a Materios preprod validator (SPO or permissioned, you do not need to be a Cardano SPO) or a permissionless attestor.
---

# Operator Guide

> **🧪 Preprod is a public testnet, not mainnet.** Everything below describes participation on Materios **preprod**, which is currently the only network running. Rewards are paid in **tMATRA** — the prefix `t` denotes test tokens. tMATRA has **no economic value**, is not exchangeable, and is not a revenue stream. Running a preprod node is for testing, learning the operator workflow, and demonstrating uptime ahead of the mainnet launch. Mainnet (with real MATRA + economic rewards) has not launched yet — see [Mainnet Roadmap](mainnet-roadmap.md) for the gating dependencies. Operators who participate now are early contributors to the network's reliability story; "rewards" you accumulate on preprod are a leaderboard / proof-of-uptime, not money.

> **🔁 v6 chain reset, 2026-04-28.** The preprod chain was reset to v6 on 2026-04-28 to fix accumulated state issues. Genesis UTXO is now `13313ea0119e0c4330f64f1809159064a371a1bbf2050b1fe13d5492280dca50#0` (Cardano preprod) and the runtime spec_version is 211+. Existing v5 registrations did not auto-carry over — if you were on v5, follow the steps below to re-onboard. New operators just follow the steps as written.

Three ways to participate in Materios preprod and earn tMATRA:

| Role | What you do | Preprod rewards (test tokens, no economic value) | Requires |
|------|-------------|---------|----------|
| **Permissioned Validator** *(recommended for new operators)* | Produce blocks + vote on finality + attest. Selected from the operator-managed permissioned-candidate list (currently 8 permissioned seats). Active in `currentCommittee` ~3 hours after your keys land on Cardano. | tMATRA: block rewards + attestation rewards. | Send your Materios pubkeys to Flux Point Studios. **No Cardano stake pool, no tADA, no 10-day wait.** |
| **SPO Validator** | Same chain duties as Permissioned Validator. Selected via Ariadne weighted by Cardano preprod-ada stake. | tMATRA: block rewards + attestation rewards. | A registered Cardano preprod stake pool with delegated stake. ~10 days for stake-snapshot to settle. |
| **Attestor** | Verify blobs and sign attestations. Doesn't run a Materios node. | tMATRA: attestation rewards only. | Nothing. One-line install. |

> **🚀 Quick path for Permissioned Validator:** generate keys with `partner-chains-node wizards generate-keys`, DM the three pubkeys to Flux Point Studios, then run [`bootstrap-validator.sh`](https://materios.fluxpointstudios.com/releases/bootstrap-validator.sh) on a Linux host. The script handles binary download, keystore seeding, systemd unit, and validator startup. Activation lands ~3 hours after we fire the Cardano-side `upsert-permissioned-candidates` tx. See [Permissioned Validator (non-SPO)](#permissioned-validator-non-spo) for the full walkthrough.

All three contribute to network security. Validators secure consensus; attestors secure data integrity (confirming that each receipt corresponds to real blob content).

> **Choosing between SPO and Permissioned Validator (preprod):** If you already run a Cardano stake pool and want to test a future second revenue stream on the same infrastructure (rewards become real on mainnet), go SPO. If you don't have a Cardano pool but you do operate cloud / home infrastructure and want to validate on Materios directly, go Permissioned. The hardware spec, chain duties, and tMATRA reward rate on preprod are identical — only the path onto the candidate list differs. Reminder: tMATRA on preprod has no economic value; mainnet rewards (real MATRA) launch later — see [Mainnet Roadmap](mainnet-roadmap.md).

Read [Node Requirements](node-requirements.md) first — it enumerates the downloads, ports, and hardware referenced below.

## Supported Environments

The validator binary is **x86_64 Linux + glibc + systemd**. Validated environments:

✅ **Supported (recommended):**
- Native Linux x86_64 with systemd (Ubuntu 22.04/24.04 LTS, Debian 12)
- Cloud VPS instances (Hetzner Cloud, DigitalOcean, Vultr, Linode, AWS EC2, GCP, Azure) running Ubuntu/Debian
- Bare-metal Linux servers
- Hyper-V / VMware / VirtualBox VMs running native Linux

🛠 **macOS — separate manual path:**
The shipped binary is x86_64-Linux-only and won't run on macOS even via Docker Desktop's amd64 emulation (Rosetta-translated libp2p drops peer connections within ~60s). The Materios MacBook validator runs today via a custom-built `aarch64-apple-darwin` binary launched via launchd, with `--reserved-only` LAN-IP bootnodes and an SSH tunnel to db-sync Postgres. **For an external macOS operator not on our home LAN, this recipe is unvalidated** — recommend running a Linux VM via UTM (free), Parallels, or VMware Fusion instead. Cert-daemon (the attestor role) DOES work via Docker Desktop on macOS — it ships as a multi-arch image.

❌ **Not supported:**
- **Windows Subsystem for Linux 2 (WSL2).** WSL2's default NAT mode breaks libp2p P2P; validator sees peers transiently then drops to 0 within ~60s. Postgres I/O on WSL2 is also too slow for the partner-chain follower's stable-boundary UTxO scan. The bootstrap script refuses to run on WSL2. If you only have a Windows host, run a real Linux VM via Hyper-V (NOT WSL2).
- **Docker Desktop running an amd64 Linux container on macOS / Windows** (same Rosetta libp2p issue as above).
- **Alpine Linux / musl-libc** — the binary is glibc-linked.

**Minimum spec:** 4 vCPU, 8 GB RAM, 40 GB SSD. Budget cloud VPS tiers (~€4.50–$10/month) are sufficient for preprod validation.

---

## Permissioned Validator (non-SPO)

You don't need a Cardano stake pool to validate Materios. The chain has 8 permissioned-candidate seats (D-parameter `(8, 0)` on preprod today, with SPO seats opening on mainnet); these are filled from a governance-managed allowlist. **Apply to be added** and you'll be eligible at the next session boundary (~3 hours), not 2 Cardano epochs.

This is the right path for operators who:
- Already run cloud / home-lab infrastructure but don't run a Cardano stake pool.
- Want to validate Materios as a primary activity (not as a sidecar to existing SPO ops).
- Prefer to avoid the Cardano-side complexity (pool pledge, KES rotation, db-sync hosting if you don't already need it).
- Are upgrading from attestor-only to validator + attestor (see [Upgrading from attestor](#upgrading-from-attestor-to-validator--attestor) below).

The chain duties, reward rate, hardware spec, and operational expectations are **identical** to the SPO Validator path. The only difference is how you get on the candidate list — and how long it takes (3 hours vs 10 days).

### 0. Pre-reqs: Linux host + Cardano stack

You need:
- A Linux host meeting the spec in [Supported Environments](#supported-environments). **Not WSL2.** A €4.50/mo Hetzner CX22, a $4 DigitalOcean droplet, or a spare Ubuntu box at home all work.
- Your own Cardano-preprod follower stack: `cardano-node` (preprod) + `cardano-db-sync` + Postgres + Ogmios. Or a hosted equivalent ([TxPipe Dolos](https://txpipe.io/) / [Demeter.run](https://demeter.run/)). The validator's mainchain-follower reads committee state from this Postgres.
- Outbound access to `https://materios.fluxpointstudios.com` (artifact downloads).
- Inbound TCP 30333 reachable from `166.70.250.197` (the bootnode dials back).

### 1. Generate Materios validator keys

Download `partner-chains-node` v1.8.0 (same binary the SPO path uses; we use it here only for key generation):

```bash
curl -sSLo partner-chains-node \
  https://github.com/input-output-hk/partner-chains/releases/download/v1.8.0/partner-chains-node-v1.8.0-x86_64-linux
chmod +x partner-chains-node
sudo mv partner-chains-node /usr/local/bin/
```

Generate the three keys via the wizard (writes JSON files into `~/materios-keys/` AND seeds the in-process keystore that the bootstrap script later picks up):

```bash
mkdir -p ~/materios-keys
cd ~/materios-keys
partner-chains-node wizards generate-keys
```

The wizard prints three pubkeys + the keystore directory location. Three files appear in `~/materios-keys/`:
- `aura.json` — sr25519, block authoring
- `grandpa.json` — ed25519, finality
- `sidechain.json` — ECDSA, partner-chain identity

**Back these up offline.** Losing them means re-applying for a permissioned slot.

Extract the three pubkeys you'll send to Flux Point Studios:

```bash
SIDECHAIN_PUB=$(jq -r '.publicKey' sidechain.json)   # 66-char hex (33-byte compressed ECDSA)
AURA_PUB=$(jq -r '.publicKey' aura.json)             # 64-char hex
GRANDPA_PUB=$(jq -r '.publicKey' grandpa.json)       # 64-char hex
```

### 2. Apply for a permissioned slot

DM Flux Point Studios in the operator Discord, or open an issue in [materios](https://github.com/Flux-Point-Studios/materios/issues) titled `Permissioned validator application: <your-label>` with this body:

```
sidechain_pub: 0x<66-char-hex>
aura_pub:      0x<64-char-hex>
grandpa_pub:   0x<64-char-hex>
label:         <your-display-name>
contact:       <discord-handle or email>
```

We will:
1. Confirm the keys decode cleanly (each must be valid hex of the right length).
2. Append your sidechain pubkey to `/tmp/v6-permissioned-candidates.txt` on our infra.
3. Submit a Cardano-side `upsert-permissioned-candidates` tx (~0.3 tADA fee, paid by us).
4. Reply with the Cardano tx hash and the expected activation window (~3 hours from the tx landing).

Stake pool ownership is **not checked**. The candidates list is governed by the partner-chains pallet, separate from Cardano stake pool registration.

### 3. Run the bootstrap script

Once we confirm your keys are on Cardano, run this on your Linux host. The script downloads the v6 binary + chain-spec, pre-seeds the keystore from your `~/materios-keys/*.json` files, writes a systemd unit, and starts the validator:

```bash
# Fetch + verify the bootstrap script
curl -fsSL https://materios.fluxpointstudios.com/releases/bootstrap-validator.sh -o bootstrap-validator.sh
curl -fsSL https://materios.fluxpointstudios.com/releases/SHA256SUMS | grep bootstrap-validator.sh
sha256sum bootstrap-validator.sh   # must match the published SHA
chmod +x bootstrap-validator.sh

# Run it (substitute your db-sync connection + your aura pubkey from step 1)
sudo -E ./bootstrap-validator.sh \
  --operator-label <your-label> \
  --db-sync 'postgres://<user>:<pw>@127.0.0.1:5432/cexplorer' \
  --ogmios ws://127.0.0.1:1337 \
  --aura-pubkey 0x<your-aura-pub>
```

The script auto-installs missing deps (`curl`, `jq`, `autossh`), refuses on unsupported environments (WSL2, Alpine, non-x86_64), generates a reverse-SSH tunnel keypair (the operator can install its pubkey on our gateway for backdoor access), and starts the validator under systemd.

### 4. Apply the v6 data snapshot

partner-chains-node v1.8.0 has a known historical-sync bug — fresh validators stall at block 0 with `Inherent error: Candidates inherent required: committee needs to be stored one epoch in advance`. The fix is to import a rocksdb snapshot from a synced node. **Do this immediately after step 3 finishes** — the bootstrap script doesn't auto-apply the snapshot:

```bash
sudo systemctl stop materios-node-spo.service
rm -rf ~/materios-preprod/data/chains/materios_preprod_v6/db

cd ~/materios-preprod
curl -fLo snapshot-v6.tar.gz \
  https://materios.fluxpointstudios.com/releases/materios_preprod_v6-data-20260428-2245.tar.gz
echo "92641adfe1f1f0c3d11d7b83a1952f6755b2b3ac65cb670a11040f41868b7bd1  snapshot-v6.tar.gz" | sha256sum -c -
tar xzf snapshot-v6.tar.gz && rm snapshot-v6.tar.gz

sudo systemctl start materios-node-spo.service
sleep 25
```

Tarball contains only `data/chains/materios_preprod_v6/db/` — no keystore, no peer-id keys, so your validator's identity stays yours.

### 5. Verify health + activation

```bash
# Best block should be well above 0 (snapshot is from block #4173) and climbing every ~6s
curl -s -X POST http://127.0.0.1:9945 -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"chain_getHeader","params":[]}' \
  | jq -r '.result.number' | xargs -I{} printf "best: 0x%s = %d\n" {} 0x{}

# Peers should be ≥1
curl -s -X POST http://127.0.0.1:9945 -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"system_health","params":[]}'
```

Activation: ~3 hours after Flux Point Studios fires the Cardano-side `upsert-permissioned-candidates` tx, your sidechain pubkey appears in `sessionCommitteeManagement.currentCommittee`. Aura assigns you slots automatically; no further action.

You can watch for activation in the [explorer](https://fluxpointstudios.com/materios/explorer) committee tab — your SS58 should appear when selected.

### Upgrading from attestor to validator + attestor

If you're already running an attestor (cert-daemon via operator-kit's `install.sh`) and want to add the validator role on top:

1. **Keep your existing attestor running.** Don't touch the cert-daemon — it'll keep earning attestation rewards on the OrinqReceipts committee. Its keys and identity are independent from the validator's.
2. **Set up the validator on a separate Linux host.** Validator + attestor on the same host is fine in principle, but most attestor installs are minimal (a single Docker container) while a validator needs the full Cardano-preprod stack + ~40GB disk. A separate VPS or repurposed Linux box keeps things clean.
3. **Generate fresh validator keys** via `partner-chains-node wizards generate-keys` (don't reuse the attestor's cert-daemon key — different scheme, different role).
4. **Follow steps 2-5 above** to apply, bootstrap, snapshot, verify.
5. **You'll now earn both reward streams** — attestation rewards from the cert-daemon (instant) + block rewards + attestation rewards from the validator (per-block + per-cert). Both pools mint tMATRA on preprod (no economic value; see [Rewards](#rewards) section).

Your attestor SS58 and validator SS58 are different. The explorer leaderboard tracks them separately.

### Operating

| Task | Command |
|---|---|
| Logs | `sudo journalctl -u materios-node-spo -f` |
| Restart | `sudo systemctl restart materios-node-spo` |
| Peer count | `curl -s -X POST http://127.0.0.1:9945 -H 'Content-Type: application/json' -d '{"jsonrpc":"2.0","id":1,"method":"system_health"}'` |
| Finalized head | `curl -s -X POST http://127.0.0.1:9945 -H 'Content-Type: application/json' -d '{"jsonrpc":"2.0","id":1,"method":"chain_getFinalizedHead"}'` |
| Status (best + finality + tunnel) | `curl -s http://127.0.0.1:9945/health` |
| Upgrade binary | Re-run `bootstrap-validator.sh` (idempotent — it'll re-download + verify the new SHA, restart the service) |

### What if you also want to register as an SPO later?

You can. Run the steps in [SPO Validator](#spo-validator) using the **same** sidechain/aura/grandpa keys. Your registration becomes eligible after the 2-epoch stake snapshot, and you'll be selectable from **either** the permissioned track or the SPO track. (Selection is mutually exclusive per session — you'll fill at most one seat per session.)

### Removing yourself from the permissioned list

DM us in the operator Discord. We submit a partner-chains tx to remove your sidechain pubkey from the candidates list. Takes effect at the next stable mc_epoch boundary. You can also just stop running the node — your seat falls back to other candidates without any tx, and we'll prune you on the next operator review.

---

## SPO Validator

Registration happens on Cardano preprod, not on Materios. Once you're registered and your stake snapshot has settled, Ariadne will select your pool with probability proportional to your active preprod-ada stake.

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
export GENESIS_UTXO="13313ea0119e0c4330f64f1809159064a371a1bbf2050b1fe13d5492280dca50#0"
export OGMIOS_URL="ws://<your-ogmios-host>:1337"
export PAYMENT_SKEY=/path/to/your/pool/payment.skey
export COLD_SKEY=/path/to/your/pool/cold.skey
```

`GENESIS_UTXO` is the Materios preprod **v6** constant; the others are yours. (Pre-v6 registrations used a different genesis UTXO and don't carry over — re-register if you registered before 2026-04-28.)

### 1. Download the Partner Chains CLI (`partner-chains-node` v1.8.0)

```bash
curl -sSLo partner-chains-node \
  https://github.com/input-output-hk/partner-chains/releases/download/v1.8.0/partner-chains-node-v1.8.0-x86_64-linux
chmod +x partner-chains-node
sudo mv partner-chains-node /usr/local/bin/
partner-chains-node --version   # → 1.8.0-...
```

This is a standalone tool for registering with any partner chain. It doesn't run a node; it just signs transactions and queries Cardano / Materios state.

> **Upstream note (2026-04):** The original `input-output-hk/partner-chains` repo was [archived 2026-04-23](https://github.com/input-output-hk/partner-chains#warning-archived) and continued development moved into [`midnightntwrk/midnight-node`](https://github.com/midnightntwrk/midnight-node) under the new name `midnight-node-toolkit`. **The v1.8.0 binary linked above remains the correct version for current Materios preprod** — Materios's runtime is built against partner-chains v1.8.0 and the archived release page still serves the binary. Don't substitute newer toolkit versions without coordination; CLI/runtime version compatibility matters. We'll update this section when Materios advances to a newer toolkit.

### 2. Generate Materios keys

```bash
mkdir -p ~/materios-keys
cd ~/materios-keys
partner-chains-node wizards generate-keys
```

The wizard prints the three pubkeys + a keystore path. Three JSON files appear in `~/materios-keys/`:
- `aura.json` (sr25519, block authoring)
- `grandpa.json` (ed25519, finality)
- `sidechain.json` (ECDSA, partner-chain identity)

Each `.json` contains mnemonic, public key (hex + SS58), and secret seed. **Back them up offline** — losing the sidechain key means re-registering from scratch.

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
  --chain /path/to/chain-spec-v6-raw.json \
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
  --chain /path/to/chain-spec-v6-raw.json \
  --mainchain-epoch <X+2>
```

Your pool ID appears in `registered_candidates` once you're eligible.

### 7. Set up the node via bootstrap-validator.sh

The same bootstrap script the Permissioned Validator path uses also works for SPO Validators — it handles binary download, keystore seeding from your `~/materios-keys/*.json` files, env-file generation, systemd unit, and validator startup. Run on your Linux host:

```bash
# Fetch + verify the bootstrap script
curl -fsSL https://materios.fluxpointstudios.com/releases/bootstrap-validator.sh -o bootstrap-validator.sh
curl -fsSL https://materios.fluxpointstudios.com/releases/SHA256SUMS | grep bootstrap-validator.sh
sha256sum bootstrap-validator.sh   # must match the published SHA
chmod +x bootstrap-validator.sh

# Run it
sudo -E ./bootstrap-validator.sh \
  --operator-label <your-pool-ticker> \
  --db-sync 'postgres://<user>:<pw>@127.0.0.1:5432/cexplorer' \
  --ogmios "$OGMIOS_URL" \
  --aura-pubkey 0x"$AURA_PUB"
```

The bootstrap script is x86_64-Linux + glibc + systemd; see [Supported Environments](#supported-environments) for what's accepted. It refuses on WSL2 (NAT breaks libp2p) and Alpine (musl mismatch).

### 8. Apply the v6 data snapshot

partner-chains-node v1.8.0 stalls at block 0 on a fresh DB (`Inherent error: Candidates inherent required`). Apply the v6 rocksdb snapshot to skip historical sync — same procedure as the Permissioned Validator path:

```bash
sudo systemctl stop materios-node-spo.service
rm -rf ~/materios-preprod/data/chains/materios_preprod_v6/db

cd ~/materios-preprod
curl -fLo snapshot-v6.tar.gz \
  https://materios.fluxpointstudios.com/releases/materios_preprod_v6-data-20260428-2245.tar.gz
echo "92641adfe1f1f0c3d11d7b83a1952f6755b2b3ac65cb670a11040f41868b7bd1  snapshot-v6.tar.gz" | sha256sum -c -
tar xzf snapshot-v6.tar.gz && rm snapshot-v6.tar.gz

sudo systemctl start materios-node-spo.service
sleep 25
```

Verify:

```bash
curl -s -X POST http://127.0.0.1:9945 \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"system_health"}'
```

`isSyncing: false` and `peers ≥ 1` before proceeding.

### 9. Confirm eligibility

Once your stake snapshot has settled (step 6) and your node is running, verify you're in the eligible candidate pool:

```bash
partner-chains-node ariadne-parameters \
  --chain ~/materios-preprod/chain-spec-v6-raw.json \
  --mainchain-epoch <current-preprod-epoch>
```

Your `sidechain_pub_key` should appear in `registered_candidates` with a `stake_pool_pub_key` matching your cold key.

After the next Materios session boundary, check the committee:

```bash
# Via any Materios RPC (yours or public)
curl -s -X POST http://127.0.0.1:9945 \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"state_getStorage","params":["<currentCommittee storage key>"]}'
```

Easier: open the [explorer](https://fluxpointstudios.com/materios/explorer) committee tab — your sidechain pubkey shows up by its SS58 derivation.

### Operating

| Task | Command |
|---|---|
| Logs | `sudo journalctl -u materios-node-spo -f` |
| Restart | `sudo systemctl restart materios-node-spo` |
| Peer count | `curl -s -X POST http://127.0.0.1:9945 -H 'content-type: application/json' -d '{"jsonrpc":"2.0","id":1,"method":"system_health"}'` |
| Finalized head | `curl -s -X POST http://127.0.0.1:9945 -H 'content-type: application/json' -d '{"jsonrpc":"2.0","id":1,"method":"chain_getFinalizedHead"}'` |
| Upgrade binary | Re-run `bootstrap-validator.sh` (idempotent — fetches new SHA-verified binary, restarts unit) |

### KES renewal

Cardano KES op-certs expire ~every 90 days on mainnet / ~9 days on preprod. Re-issue the op-cert with a fresh KES period and restart your cardano-node. **No Materios action is needed** — your partner-chain registration is independent of KES. As long as your Cardano pool is minting blocks, your delegation stays active.

### Rotating Materios keys

If you ever need to rotate your Materios keys (compromised host, etc.), submit a new `smart-contracts register` with the new keys. The old registration remains until you consume a UTXO to deregister (`smart-contracts deregister`). Wait 2 epochs for the new snapshot before expecting selection with the rotated keys.

### What happens if you go offline?

- **Selected for the committee but offline:** the slots you'd have minted go unclaimed; your Grandpa vote is simply absent. No slashing on Materios preprod. Finality stays healthy as long as 2f+1 of the committee stays online (that's 2-of-3 for 3 permissioned + 2 registered at the current D parameter).
- **Missed a whole epoch:** zero block rewards for that epoch. No penalty on future selection.
- **Pool depledged / Cardano-side issues:** your partner-chain registration stays on-chain but you drop out of Ariadne's weighted pool once the Cardano snapshot stops including you. Re-register normally once your Cardano pool is healthy.

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
| OS | Linux, macOS (Docker Desktop), or Windows (WSL2 + Docker Desktop) |

> **Note:** WSL2 / Docker Desktop is fine for the **attestor** role (cert-daemon is a multi-arch image, pure Python, libp2p-insensitive). It is **NOT** supported for the validator role — see [Supported Environments](#supported-environments) above.

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

### Full Validator from operator-kit (deprecated)

Historically operator-kit shipped a `--mode validator` that provisioned a Substrate node and requested addition to the permissioned-candidate list. That mode is **retired** as of v6 (2026-04-28) — the canonical path is now [`bootstrap-validator.sh`](#permissioned-validator-non-spo) with explicit operator control over Linux host, db-sync provisioning, and key custody. If you're an existing operator-kit `--mode validator` user, your setup did not survive the v6 reset; follow the [Permissioned Validator (non-SPO)](#permissioned-validator-non-spo) path to re-onboard on v6.

### Upgrading from attestor to validator + attestor

Already running an attestor and want to add the validator role on top? The cert-daemon stays as-is; you spin up a separate Linux host for the validator and follow the [Permissioned Validator](#permissioned-validator-non-spo) path. Full walkthrough in that section under [Upgrading from attestor](#upgrading-from-attestor-to-validator--attestor).

---

## Rewards

> **🧪 Preprod is not mainnet. Rewards on this network are tMATRA — testnet tokens with no economic value.** The pool sizes below describe the *mainnet* tokenomics design (where rewards become real MATRA). On preprod, those same pools mint tMATRA at the same rates so operators can preview behavior, but tMATRA is not exchangeable, not a revenue stream, and not redeemable for anything. See [Mainnet Roadmap](mainnet-roadmap.md) for what gates the real-rewards transition.

| Pool | Reserve (mainnet design) | Reward | Who earns (on preprod, paid in tMATRA) |
|---|---|---|---|
| Block production | 150M MATRA (15% of total supply) | Per-block minter credit, distributed at era end | All Validators (SPO + Permissioned) |
| Attestation | 50M MATRA (5%) | 10 per certification per signer, paid instantly | All signers (Validators + Attestors) |

Era length is ~24 hours (14,400 blocks). Block reward is proportional to blocks minted per era — better uptime = bigger share. No slashing for downtime on preprod. Both validator paths (SPO and Permissioned) earn at the same rate per block produced.

**What preprod participation IS for:**
- Proving uptime + reliability before mainnet (operators with strong preprod track records will be prioritized for mainnet onboarding)
- Testing your operational stack (key management, monitoring, watchdogs, upgrades)
- Earning leaderboard / proof-of-uptime credit
- Helping us harden the network before real value moves through it

**What preprod participation IS NOT:**
- A revenue stream
- A way to "farm" tokens with future value
- Convertible to ADA or any other asset

---

## Health Endpoints

Cert-daemon local HTTP (port 8080):

| Path | Purpose |
|---|---|
| `GET /health` | Liveness. 200 if process is up. |
| `GET /ready` | Readiness. 200 if connected to chain + polled recently. |
| `GET /status` | Full state: height, finality gap, pending receipts, uptime, version. |
| `GET /metrics` | Prometheus. |

Node RPC (9945, localhost): standard Substrate RPC surface. `system_health`, `chain_getHeader`, `chain_getFinalizedHead`, `state_getStorage`, `author_insertKey`, etc.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Validator stuck at block #0 with `Inherent error: Candidates inherent required: committee needs to be stored one epoch in advance` | Known partner-chains-1.8.0 historical-sync bug | Apply the v6 data snapshot per Permissioned Validator [step 4](#4-apply-the-v6-data-snapshot) |
| Validator at peers=0 immediately after start | Running on WSL2 / Docker Desktop on macOS / behind aggressive NAT | See [Supported Environments](#supported-environments). WSL2 NAT breaks libp2p; move to native Linux or a Linux VM via Hyper-V/UTM. |
| Validator at peers=0 on a real Linux host | Inbound TCP 30333 not reachable from `166.70.250.197` | Open port 30333/tcp on your firewall (UFW + cloud-provider security group); confirm with `nc -zv <your-public-ip> 30333` from a different network. |
| `registration-signatures` errors on `mainchain-signing-key` | Didn't strip the 5820 CBOR prefix | `jq -r '.cborHex' cold.skey \| cut -c5-` |
| `smart-contracts register` fails with `UTxO already spent` | Your `$REGISTRATION_UTXO` was consumed between prep + submit | Pick a fresh UTXO; re-sign (same sidechain / spo keys are fine). |
| Node starts then crashes on first MC epoch boundary | Wrong genesis UTXO env or v5 binary on v6 chain | Verify `PARTNER_CHAIN_GENESIS_UTXO=13313ea0119e0c4330f64f1809159064a371a1bbf2050b1fe13d5492280dca50#0`; re-run `bootstrap-validator.sh` to pull the v6 binary. |
| `registration-status` says "not registered" 10 minutes after submit | Cardano preprod hasn't included the tx yet | Wait for 2 blocks + check the txhash on [preprod.cexplorer.io](https://preprod.cexplorer.io/) |
| Registered but `ariadne-parameters` doesn't list you | Stake snapshot hasn't rolled forward yet | You need to be past epoch X+2; see step 6 |
| In `registered_candidates` but never selected | Stake too low relative to other pools | Delegation volume determines selection probability; grow your pool's stake or wait for variance |
| Cert-daemon `Heartbeat rejected: 403 {"error":"Validator not registered"}` | Gateway's api_keys registry doesn't have your SS58 (e.g. after a chain reset) | Contact Flux Point Studios to re-register your SS58; this is a known recovery step across resets. |
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
│  materios-node-spo  ── P2P :30333 ──→ Materios net│
│    ├─ reads db-sync Postgres                      │
│    ├─ reads Ogmios (for SPO CLI only)             │
│    └─ RPC :9945 (localhost)                       │
│                                                    │
│  cert-daemon  ─── WSS → Materios RPC               │
│    └─ attests + earns tMATRA per cert             │
└────────────────────────────────────────────────────┘
```

### Permissioned Validator

```
┌─────────────── Your validator host ──────────────┐
│  cardano-db-sync (Postgres) — yours OR managed   │
│    (TxPipe / Demeter / Blockfrost — no Cardano   │
│     stake pool needed)                            │
│  ogmios — points at any Cardano preprod node     │
│                                                    │
│  materios-node-spo  ── P2P :30333 ──→ Materios net│
│    ├─ reads db-sync Postgres                      │
│    └─ RPC :9945 (localhost)                       │
│                                                    │
│  cert-daemon (optional, attestor add-on)          │
│    └─ WSS → Materios RPC, earns tMATRA per cert   │
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
