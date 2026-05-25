---
description: Trustless registration recipe — bind a Cardano preprod stake pool to a Materios sidechain identity, get selected by Ariadne, and run a validator.
---

# SPO Onboarding (Preprod)

> **Preprod is a public testnet.** Rewards are paid in **tMATRA** (no economic value). See [Mainnet Roadmap](mainnet-roadmap.md) for the path to real-MATRA rewards.

This recipe takes a Cardano preprod SPO from zero to a producing Materios validator. No FPS approval is required; selection is driven by Ariadne over your Cardano-side stake.

## Prerequisites

- A **registered Cardano preprod stake pool**. If you don't have one yet, follow the [official Cardano SPO course](https://cardano-course.gitbook.io/cardano-course/handbook) on preprod (testnet-magic 1) before continuing. You'll come back here with `cold.skey`, `vrf.skey`, `kes.skey`, `op.cert`, a payment address, and a registered pool ID.
- **~100 tADA** in the pool's payment address (~3 tADA for the registration UTXO + buffer for fees).
- **Linux host:** 4 vCPU, 8 GB RAM, NVMe SSD, ≥ 100 GB free, Ubuntu 22.04/24.04 LTS or Debian 12, inbound TCP 30333 reachable from the internet. Not WSL2, not Alpine, not Docker Desktop's amd64 emulation. See [Operator Guide → Hardware](operator-guide.md#hardware) for why.
- **Outbound** HTTPS to `materios.fluxpointstudios.com` and `github.com`.

You'll also need a synced Cardano preprod stack:

- `cardano-node` on preprod (testnet-magic `1`)
- `cardano-db-sync` writing to Postgres
- `ogmios` exposing `ws://<host>:1337`

A managed provider (TxPipe Dolos, Demeter.run, Blockfrost) is an acceptable substitute for db-sync + Ogmios; you still need a local `cardano-cli`-capable wallet for the registration tx.

## Chain parameters

These values are stable for the lifetime of the v6 preprod chain. Hard-code them in your scripts.

| Field | Value |
|---|---|
| Cardano network | preprod (testnet-magic `1`) |
| **Genesis UTXO** | `13313ea0119e0c4330f64f1809159064a371a1bbf2050b1fe13d5492280dca50#0` |
| Partner-chain genesis hash | `0x0e46e33f639a56cc8780fd871d9a15e16d99af248526f907cb560cb40849f7bf` |
| Governance authority hash | `0x680a93dd4deb4873fc0aa31678eb02c57258717953ffc9a654b0af78` (single key, threshold 1) |
| CommitteeCandidate validator | `addr_test1wrld9uhaepas48twjy3qevncsyrhjdqnkz2wzu4yzjc2qhq24f4v4` |
| PermissionedCandidates validator | `addr_test1wzyzwx0kcdgs2hc8t5w0d3g4l7s2qhvv2qcyws0m8sypwxgghu099` |
| D-parameter | `(5, 2)` — 5 permissioned + 2 registered |
| Chain spec | `https://materios.fluxpointstudios.com/releases/chain-spec-v6-raw.json` |
| Latest data snapshot | `https://materios.fluxpointstudios.com/operator-snapshots/preprod/latest.json` |
| Public bootnode | `/ip4/166.70.250.197/tcp/30333/p2p/12D3KooWPueKoxRAirTTKH4Y2qQAsJDegWMjS4k89Z7izCbZKgkM` |

Convenience env vars (used throughout):

```bash
export GENESIS_UTXO="13313ea0119e0c4330f64f1809159064a371a1bbf2050b1fe13d5492280dca50#0"
export OGMIOS_URL="ws://127.0.0.1:1337"
export PAYMENT_SKEY=/path/to/your/pool/payment.skey
export COLD_SKEY=/path/to/your/pool/cold.skey
```

## 1. Provision Postgres for cardano-db-sync

Partner-chains queries db-sync's Postgres on every block import. The defaults are too small, and one missing index will stall your validator.

Create the partner-chains index before starting materios-node:

```bash
psql "$DB_SYNC_CONNECTION_STRING" -c \
  "CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_ma_tx_out_ident ON ma_tx_out(ident);"
```

The index takes 2-5 min to build on a fresh db-sync. If materios-node starts before it completes, every block import takes 2.5s+ and your peers will drop you.

Apply the tuning baseline (`/etc/postgresql/15/main/postgresql.conf`, adjust path for your version):

```
shared_buffers = 2GB
effective_cache_size = 6GB
work_mem = 64MB
maintenance_work_mem = 1GB
random_page_cost = 1.1
effective_io_concurrency = 200
```

Then `sudo systemctl restart postgresql && psql "$DB_SYNC_CONNECTION_STRING" -c "ANALYZE;"`. Scale up proportionally on larger hosts.

Full background, diagnostics, and slow-sync recovery: [OPERATOR_KIT.md → Postgres prerequisites](https://github.com/Flux-Point-Studios/materios/blob/main/docs/OPERATOR_KIT.md#postgres-prerequisites-for-cardano-db-sync).

## 2. Download the partner-chains CLI

The IOG `partner-chains-node` v1.8.0 binary handles key generation, signature production, and registration submission. Materios's runtime is pinned to v1.8.0 — do not substitute a newer toolkit version.

```bash
curl -sSLo partner-chains-node \
  https://github.com/input-output-hk/partner-chains/releases/download/v1.8.0/partner-chains-node-v1.8.0-x86_64-linux
chmod +x partner-chains-node
sudo mv partner-chains-node /usr/local/bin/
partner-chains-node --version   # → 1.8.0-...
```

## 3. Generate Materios validator keys

The `wizards generate-keys` flow creates one 24-word mnemonic and derives all three Substrate keys from it (ECDSA sidechain, sr25519 aura, ed25519 grandpa), seeding the node keystore in one shot.

```bash
mkdir -p ~/materios-keys && cd ~/materios-keys
partner-chains-node wizards generate-keys
```

When the wizard prompts for a base path, answer `./data`. It writes three JSON files into `~/materios-keys/` and prints the three pubkeys:

```
sidechain (ECDSA)  : 0x<66-char-hex>
aura  (sr25519)    : 0x<64-char-hex>
grandpa (ed25519)  : 0x<64-char-hex>
```

**Back up the mnemonic offline immediately.** Losing it means re-registering on Cardano.

Extract the hexes you'll pass to the next steps:

```bash
SIDECHAIN_PUB=$(jq -r '.publicKey' sidechain.json)   # 66-char hex
AURA_PUB=$(jq -r '.publicKey' aura.json)             # 64-char hex
GRANDPA_PUB=$(jq -r '.publicKey' grandpa.json)       # 64-char hex
```

### Manual sidechain-key derivation

If you'd rather derive the secp256k1 key yourself (e.g. importing an existing 32-byte hex secret), the public key is the compressed-point encoding:

```python
from cryptography.hazmat.primitives.asymmetric import ec
from cryptography.hazmat.primitives import serialization

sk_hex = "<your 32-byte secret hex, no 0x prefix>"
sk = ec.derive_private_key(int(sk_hex, 16), ec.SECP256K1())
pk = sk.public_key().public_bytes(
    encoding=serialization.Encoding.X962,
    format=serialization.PublicFormat.CompressedPoint,
)
print("0x" + pk.hex())   # 33-byte compressed pubkey, 66 hex chars
```

The Substrate keystore wants the secret as the `secretSeed` field of `sidechain.json` (`0x`-prefixed 32-byte hex) and the corresponding public as the `publicKey` field. Write both before starting materios-node.

## 4. Pick a registration UTXO

The Cardano-side registration tx consumes one UTXO from your pool's payment address as a uniqueness nonce.

```bash
cardano-cli conway query utxo \
  --testnet-magic 1 \
  --address $(cat /path/to/pool/payment.addr) \
  --output-json | jq -r 'to_entries[0].key'

# Pick any UTXO with ≥ 3 tADA:
export REGISTRATION_UTXO="<txhash>#<ix>"
```

If your address has only one large UTXO, split off a small one first (`cardano-cli conway transaction build` against your own address).

## 5. Sign the registration

This step is fully offline-capable: it produces signatures from your Cardano cold key and your sidechain ECDSA key, bound to the `GENESIS_UTXO` + `REGISTRATION_UTXO` pair so they can't be replayed.

Strip the CBOR prefix from your cold key (cardano-cli wraps the 32-byte scalar in a CBOR `5820` byte-string):

```bash
COLD_SKEY_RAW=$(jq -r '.cborHex' "$COLD_SKEY" | sed 's/^5820//')
SIDECHAIN_SKEY=$(jq -r '.secretSeed' ~/materios-keys/sidechain.json | sed 's/^0x//')

partner-chains-node registration-signatures \
  --genesis-utxo "$GENESIS_UTXO" \
  --registration-utxo "$REGISTRATION_UTXO" \
  --mainchain-signing-key "$COLD_SKEY_RAW" \
  --sidechain-signing-key "$SIDECHAIN_SKEY" \
  > phase-a.json

cat phase-a.json
```

Output:

```json
{
  "spo_public_key":       "0x...",
  "spo_signature":        "0x...",
  "sidechain_public_key": "0x...",
  "sidechain_signature":  "0x..."
}
```

## 6. Submit the registration

```bash
SPO_PUB=$(jq -r '.spo_public_key'        phase-a.json)
SPO_SIG=$(jq -r '.spo_signature'         phase-a.json)
SIDE_SIG=$(jq -r '.sidechain_signature'  phase-a.json)

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

Successful output ends with `Transaction submitted. ID: <txhash>`. Record the txhash; you'll confirm it on Cardano next.

Verify the tx made it on-chain:

```bash
curl -fsSL https://materios.fluxpointstudios.com/releases/chain-spec-v6-raw.json \
  -o ~/chain-spec-v6-raw.json

partner-chains-node registration-status \
  --chain ~/chain-spec-v6-raw.json \
  --mainchain-pub-key "$SPO_PUB" \
  --mainchain-epoch <current-preprod-epoch>
```

The current preprod epoch is on the front page of [preprod.cexplorer.io](https://preprod.cexplorer.io/). The chain spec stays in place for step 8.

## 7. Restore the latest data snapshot

partner-chains-node v1.8.0 has a known historical-sync bug: a fresh node stalls at block 0 with `Inherent error: Candidates inherent required`. The fix is to import a rocksdb snapshot from a synced node.

The latest snapshot is published hourly; older ones are pruned after 48h. Read the manifest, restore, verify:

```bash
mkdir -p ~/materios-preprod/data/chains
cd ~/materios-preprod

# Fetch the manifest and pull the current tarball
MANIFEST=$(curl -fsSL https://materios.fluxpointstudios.com/operator-snapshots/preprod/latest.json)
TARBALL_URL=$(echo "$MANIFEST" | jq -r .url)
EXPECTED_SHA=$(echo "$MANIFEST" | jq -r .sha256)

curl -fLo snapshot.tar.gz "$TARBALL_URL"
echo "${EXPECTED_SHA}  snapshot.tar.gz" | sha256sum -c -

tar xzf snapshot.tar.gz && rm snapshot.tar.gz
```

The tarball contains only `data/chains/materios_preprod_v6/db/` — no keystore, no peer-id keys, so your validator identity stays yours.

## 8. Start materios-node

The bootstrap script handles binary download, SHA verification, keystore wiring, snapshot restore (if not already done), and systemd unit:

```bash
curl -fsSL https://materios.fluxpointstudios.com/releases/bootstrap-validator.sh -o bootstrap-validator.sh
curl -fsSL https://materios.fluxpointstudios.com/releases/SHA256SUMS | grep bootstrap-validator.sh
sha256sum bootstrap-validator.sh   # must match
chmod +x bootstrap-validator.sh

sudo -E ./bootstrap-validator.sh \
  --operator-label <your-pool-ticker> \
  --db-sync 'postgres://<user>:<pw>@127.0.0.1:5432/cexplorer' \
  --ogmios "$OGMIOS_URL" \
  --aura-pubkey 0x"$AURA_PUB"
```

If you build your own systemd unit instead, the validator must run with:

```
materios-node-spo \
  --chain ~/materios-preprod/chain-spec-v6-raw.json \
  --base-path ~/materios-preprod/data \
  --validator \
  --name <your-pool-ticker> \
  --port 30333 \
  --rpc-port 9945 \
  --public-addr /ip4/<YOUR.PUBLIC.IP>/tcp/30333 \
  --bootnodes /ip4/166.70.250.197/tcp/30333/p2p/12D3KooWPueKoxRAirTTKH4Y2qQAsJDegWMjS4k89Z7izCbZKgkM
```

`--public-addr` is required for SPOs. Without it, peers can't dial you back and the blocks you author won't gossip.

Verify health:

```bash
curl -s -X POST http://127.0.0.1:9945 \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"system_health"}'
```

Expect `isSyncing: false` and `peers ≥ 1` within a couple of minutes of starting (snapshot floor is the published `head_block_at_publish`; you should sync past it to the live tip).

## 9. Wait for the stake snapshot

Ariadne reads the **2-epoch-stable** Cardano stake snapshot. Your registration becomes selectable two preprod epochs (~10 days) after the epoch it was included in.

| Cardano epoch | What's happening |
|---|---|
| E | Registration tx included. |
| E+1 | Stake snapshot captured (`mark`). |
| E+2 | Snapshot becomes `set` → Ariadne considers you from this epoch onward. |

Watch the queue:

```bash
partner-chains-node ariadne-parameters \
  --chain ~/materios-preprod/chain-spec-v6-raw.json \
  --mainchain-epoch <E+2>
```

Your `sidechain_pub_key` appears in `registered_candidates` once you're eligible.

## 10. Verify selection

At the next Materios `mc_epoch` boundary after your snapshot becomes `set`, Ariadne runs a fresh committee draw. With the current D-parameter `(5, 2)` you compete with other SPOs for 2 registered seats; probability is proportional to your active delegated stake.

Watch the [explorer Committee tab](https://fluxpointstudios.com/materios/explorer) — your SS58 (derived from your sidechain pubkey) shows up when selected.

Or query directly:

```bash
curl -s -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"state_call","params":["SessionCommitteeManagementApi_get_current_committee","0x"]}' \
  https://materios.fluxpointstudios.com/preprod-rpc | jq .
```

Once you're in `currentCommittee`, Aura assigns slots automatically. Your validator starts producing blocks ~6s per assigned slot; rewards accrue per block + per attestation.

## 11. Operating

| Task | Command |
|---|---|
| Logs | `sudo journalctl -u materios-node-spo -f` |
| Restart | `sudo systemctl restart materios-node-spo` |
| Peer count | `curl -s -X POST http://127.0.0.1:9945 -H 'content-type: application/json' -d '{"jsonrpc":"2.0","id":1,"method":"system_health"}'` |
| Finalized head | `curl -s -X POST http://127.0.0.1:9945 -H 'content-type: application/json' -d '{"jsonrpc":"2.0","id":1,"method":"chain_getFinalizedHead"}'` |
| Upgrade binary | Re-run `bootstrap-validator.sh` (idempotent — fetches the new SHA-verified binary, restarts the unit) |

### KES renewal

Cardano KES op-certs expire ~every 9 days on preprod. Re-issue with a fresh KES period and restart your `cardano-node`; no Materios-side action is needed (your partner-chain registration is independent of KES).

```bash
CURRENT_KES=$(cardano-cli conway query tip --testnet-magic 1 \
  --socket-path /ipc/node.socket | jq -r '.slot / 129600 | floor')

cardano-cli conway node issue-op-cert \
  --kes-verification-key-file kes.vkey \
  --cold-signing-key-file cold.skey \
  --operational-certificate-issue-counter-file cold.counter \
  --kes-period "$CURRENT_KES" \
  --out-file op.cert
```

### Rotating Materios keys

Submit a fresh `smart-contracts register` with new keys. The old registration stays on-chain until you call `smart-contracts deregister`. Wait 2 epochs after the new tx before expecting selection with the rotated keys.

### Going offline

If selected but offline, the slots you would have minted go unclaimed and your Grandpa vote is absent — no slashing on preprod. Finality stays healthy while 2f+1 of the committee is online. Drop out of Ariadne's pool entirely by stopping your Cardano pool (depledge, missed snapshot) or calling `smart-contracts deregister`.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Node stuck at block 0, log says `Inherent error: Candidates inherent required` | Fresh DB without snapshot | Apply the snapshot per [step 7](#7-restore-the-latest-data-snapshot) |
| Snapshot restore succeeds but node stalls at the snapshot floor, peers loop on "Banned, disconnecting" | Stale libp2p peer or slow db-sync | See [OPERATOR_KIT.md → Sync stuck at snapshot floor](https://github.com/Flux-Point-Studios/materios/blob/main/docs/OPERATOR_KIT.md#sync-stuck-at-snapshot-floor--peer-ban-loop) — `--reserved-only` to the FPS bootnode is the immediate workaround |
| `sqlx::query: slow statement ... elapsed=2.5s` in node logs + repeated peer drops | Postgres missing `idx_ma_tx_out_ident` or untuned | Re-run [step 1](#1-provision-postgres-for-cardano-db-sync) |
| `registration-signatures` errors on `mainchain-signing-key` | Didn't strip the `5820` CBOR prefix from cold.skey | `jq -r '.cborHex' cold.skey \| sed 's/^5820//'` |
| `smart-contracts register` fails with `UTxO already spent` | Your `$REGISTRATION_UTXO` was consumed between prep + submit | Pick a fresh UTXO; re-sign (same sidechain / SPO keys are fine) |
| `registration-status` says "not registered" 10 min after submit | Cardano hasn't included your tx yet | Wait 2 Cardano blocks; check the txhash on [preprod.cexplorer.io](https://preprod.cexplorer.io/) |
| In `registered_candidates` but never selected | Stake too low vs other pools, or D-parameter `R=0` | Grow your pool's stake or wait for variance; D-parameter is `(5, 2)` today |
| Validator at peers=0 on a real Linux host | Inbound TCP 30333 unreachable | Open 30333/tcp on your firewall + cloud security group; confirm with `nc -zv <your-public-ip> 30333` from another network |
| Finality gap > 10 in explorer authority-lag panel | Committee under-quorum or your node behind | Compare your `chain_getFinalizedHead` to the explorer's — if you're the lagger, restart; if the chain is the lagger, check Discord for an active incident |

Full sync diagnostics, RPC reference, and recovery playbooks: [OPERATOR_KIT.md](https://github.com/Flux-Point-Studios/materios/blob/main/docs/OPERATOR_KIT.md).

## Getting help

- Materios Discord: `#spo-support`
- Chain explorer: [fluxpointstudios.com/materios/explorer](https://fluxpointstudios.com/materios/explorer)
- Issue tracker: [Flux-Point-Studios/materios-operator-kit](https://github.com/Flux-Point-Studios/materios-operator-kit/issues)
- Preprod faucet: [docs.cardano.org/cardano-testnets/tools/faucet](https://docs.cardano.org/cardano-testnets/tools/faucet)
