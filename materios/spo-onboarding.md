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
| D-parameter | `(15, 1)` — 15 permissioned + 1 registered |
| Chain spec | `https://materios.fluxpointstudios.com/releases/chain-spec-v6-raw.json` |
| Latest data snapshot | `https://materios.fluxpointstudios.com/operator-snapshots/preprod/latest.json` |
| Public bootnode | `/dns4/bootnode.materios.fluxpointstudios.com/tcp/30333/p2p/12D3KooWPueKoxRAirTTKH4Y2qQAsJDegWMjS4k89Z7izCbZKgkM` |

Convenience env vars (used throughout):

```bash
export GENESIS_UTXO="13313ea0119e0c4330f64f1809159064a371a1bbf2050b1fe13d5492280dca50#0"
export CANDIDATES_ADDR="addr_test1wrld9uhaepas48twjy3qevncsyrhjdqnkz2wzu4yzjc2qhq24f4v4"
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

### Verify the registration landed

Registration is **immediate on Cardano L1**. The moment your tx is in a block you are registered — there is no pending state, and nothing on the Materios side has to accept it. The ~2-epoch wait everyone hears about is Ariadne **seating** ([step 9](#9-wait-for-the-stake-snapshot)), which is a separate thing that happens after you're already registered.

Your registration is a UTxO at the CommitteeCandidate validator (`$CANDIDATES_ADDR`, from [Chain parameters](#chain-parameters)) whose inline datum embeds your `spo_public_key`.

**What the checks below do and don't tell you.** Each one answers exactly this: *is there an unspent UTxO at the candidates address whose datum contains my public key?* That is the thing that goes wrong in practice — a tx that never landed, or a registration you later replaced — so a clean result is the signal you want before you start waiting on [step 9](#9-wait-for-the-stake-snapshot).

It is not a validity proof. The candidates address is a **permissionless script address**: anyone can pay a UTxO there carrying any datum they like, and these checks only substring-match your key inside the datum bytes. They do not verify your SPO or sidechain signatures, do not confirm the datum is well-formed, and do not confirm Ariadne will accept the registration. Only the chain decides that, and it tells you by seating you. If all three checks look right and you are still unseated well past E+2, the [troubleshooting table](#troubleshooting) is the next stop — not a re-registration.

> **Don't use `partner-chains-node registration-status` or `ariadne-parameters`.** Both fail on Materios with:
>
> ```
> Application(Execution(Other("Exported method CandidateValidationApi_validate_registered_candidate_data is not found")))
> ```
>
> The reason is specific, and worth stating precisely because the obvious sanity-check appears to contradict it. Neither command talks to your running node. Both take `--chain <CHAIN_SPEC>`, build a throwaway **genesis** state from the WASM embedded in that chain spec, and call the runtime API there. The genesis runtime inside `chain-spec-v6-raw.json` does not export `CandidateValidationApi`, so the call fails before any registration is examined.
>
> The **live** runtime is a different binary and *does* export it — a runtime upgrade added it after genesis. So if you query the chain directly (`state_getRuntimeVersion`, or the exports of the on-chain `:code`) you will find `CandidateValidationApi` present and conclude these commands should work. They still won't: they never load the live runtime. Fixing this needs a re-issued chain spec, not a runtime upgrade.
>
> Your registration and your odds of selection are unaffected either way — the running node validates candidates over its own internal path, not this CLI. Use a check below instead.

**A — your own db-sync** (you already run one, [step 1](#1-provision-postgres-for-cardano-db-sync)):

```bash
SPO_PUB_HEX=$(jq -r '.spo_public_key' phase-a.json | sed 's/^0x//')

psql "$DB_SYNC_CONNECTION_STRING" -X -v ON_ERROR_STOP=1 \
  -v addr="$CANDIDATES_ADDR" -v spo="$SPO_PUB_HEX" <<'SQL'
SELECT encode(tx.hash, 'hex') || '#' || txo.index AS registration_utxo,
       NOT EXISTS (SELECT 1 FROM tx_in ti
                    WHERE ti.tx_out_id    = txo.tx_id
                      AND ti.tx_out_index = txo.index) AS unspent
  FROM tx_out txo
  JOIN tx      ON tx.id = txo.tx_id
  JOIN datum d ON d.id  = txo.inline_datum_id
 WHERE txo.address = :'addr'
   AND encode(d.bytes, 'hex') LIKE '%' || :'spo' || '%';
SQL
```

One row with `unspent = t` is the answer you want: registered, current. Zero rows means the tx is not on Cardano — look up your txhash on [preprod.cexplorer.io](https://preprod.cexplorer.io/). `unspent = f` means that registration was later replaced or deregistered.

**B — Koios**, if you use a managed Cardano provider and have no local db-sync:

```bash
curl -s -X POST https://preprod.koios.rest/api/v1/address_utxos \
  -H 'content-type: application/json' \
  -d "{\"_addresses\":[\"$CANDIDATES_ADDR\"],\"_extended\":true}" \
| jq -r --arg k "$(jq -r '.spo_public_key' phase-a.json | sed 's/^0x//')" \
     '.[] | select(((.inline_datum.bytes) // "") | contains($k)) | "\(.tx_hash)#\(.tx_index)"'
```

One line back means you are registered and that UTxO is current. No output means no unspent registration carrying your key.

The `// ""` is load-bearing — don't drop it. The candidates address is permissionless, so a UTxO posted by anyone else may carry no inline datum at all; without the fallback `jq` aborts the whole pipeline on that entry (`null and string cannot have their containment checked`) and, if the datum-less UTxO sorts before yours, prints **nothing at all** — a false "not registered" for a perfectly good registration.

Note the scope: `address_utxos` returns only **unspent** UTxOs, so this check cannot distinguish "you never registered" from "your registration was spent — replaced or deregistered". Both look like empty output. If you get no output and you know your tx landed on Cardano, use check **A** (its `unspent` column tells the two apart) or **C**, or look the txhash up on [preprod.cexplorer.io](https://preprod.cexplorer.io/) and see whether the output has been consumed.

**C — the explorer**, no tooling at all. Open

```
https://materios.fluxpointstudios.com/materios/explorer/spo-journey/<SIDECHAIN_PUB>
```

with your `0x`-prefixed sidechain pubkey from [step 3](#3-generate-materios-validator-keys). It runs check A server-side and renders the whole path — registered → selected → authoring → liveness → finality — so it also answers steps 9 and 10.

Download the chain spec now; step 8 needs it in place:

```bash
curl -fsSL https://materios.fluxpointstudios.com/releases/chain-spec-v6-raw.json \
  -o ~/chain-spec-v6-raw.json
```

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

The bootstrap script handles binary download, SHA verification, keystore wiring, snapshot restore (if not already done), and systemd unit. The snapshot restore in [step 7](#7-restore-the-latest-data-snapshot) is the authoritative manual fallback — if the script's restore is ever skipped or fails, your node will genesis-replay and stall at block 0, so do step 7 by hand and re-run with `--skip-snapshot`:

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
  --bootnodes /dns4/bootnode.materios.fluxpointstudios.com/tcp/30333/p2p/12D3KooWPueKoxRAirTTKH4Y2qQAsJDegWMjS4k89Z7izCbZKgkM
```

`--public-addr` is required for SPOs. Without it, peers can't dial you back and the blocks you author won't gossip.

Verify health:

```bash
curl -s -X POST http://127.0.0.1:9945 \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"system_health"}'
```

Expect `isSyncing: false` and `peers ≥ 1` within a couple of minutes of starting (snapshot floor is the published `head_block_at_publish`; you should sync past it to the live tip).

### Post-sync divergence self-check

Once synced, confirm you landed in the **canonical** GRANDPA room, not a divergent fork. Fetch the network's finalized `{number, hash}` and compare your local block hash at that height:

```bash
NET=$(curl -fsSL https://materios.fluxpointstudios.com/chain-info)
H=$(echo "$NET" | jq -r .finalized_block)
HNET=$(echo "$NET" | jq -r .finalized)   # network's finalized hash at H

# Ask YOUR node for its block hash at the network's finalized height
HLOCAL=$(curl -s -X POST http://127.0.0.1:9945 \
  -H 'content-type: application/json' \
  -d "{\"jsonrpc\":\"2.0\",\"id\":1,\"method\":\"chain_getBlockHash\",\"params\":[$H]}" | jq -r .result)

if [ "$HLOCAL" = "$HNET" ]; then
  echo "CANONICAL room ✓ (local $H == network $HNET)"
else
  echo "DIVERGENT room ✗ — local $HLOCAL != network $HNET; re-restore from the snapshot (step 7) and restart"
fi
```

Compare at the **network's** finalized height — your local node may still be a few blocks behind. A `null` local result just means you haven't reached `H` yet; wait and re-run. A non-null hash that differs is a real fork: wipe `data/chains/materios_preprod_v6/db/`, re-restore the snapshot, and restart.

## 9. Wait for the stake snapshot

Ariadne reads the **2-epoch-stable** Cardano stake snapshot. Your registration becomes selectable two preprod epochs (~10 days) after the epoch it was included in.

| Cardano epoch | What's happening |
|---|---|
| E | Registration tx included. |
| E+1 | Stake snapshot captured (`mark`). |
| E+2 | Snapshot becomes `set` → Ariadne considers you from this epoch onward. |

There is nothing to submit and nothing to retry during this window. Your registration is already final on L1 — re-run [the verify check](#verify-the-registration-landed) any time to confirm it's still there and unspent. All that changes at E+2 is that Ariadne starts counting you.

To watch the transition, open your [SPO journey page](#verify-the-registration-landed) (check C) and wait for the **selected** milestone to light up.

`ariadne-parameters` fails the same way `registration-status` does — it runs the same genesis runtime from the chain spec, which doesn't export `CandidateValidationApi`. See the note in [step 6](#verify-the-registration-landed).

## 10. Verify selection

At the next Materios `mc_epoch` boundary after your snapshot becomes `set`, Ariadne runs a fresh committee draw. With the current D-parameter `(15, 1)` you compete with other SPOs for 1 registered seat; probability is proportional to your active delegated stake.

Watch the [explorer Committee tab](https://fluxpointstudios.com/materios/explorer) — your SS58 (derived from your sidechain pubkey) shows up when selected.

Or query directly. The call returns the committee SCALE-encoded, so match your own sidechain pubkey against it:

```bash
COMMITTEE=$(curl -s -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"state_call","params":["SessionValidatorManagementApi_get_current_committee","0x"]}' \
  https://materios.fluxpointstudios.com/preprod-rpc | jq -r .result)

case "$COMMITTEE" in
  *"${SIDECHAIN_PUB#0x}"*) echo "SEATED — you are in the current committee" ;;
  *)                       echo "not seated yet" ;;
esac
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

### Show your stats in the explorer (optional)

Your validator shows a green **Producing** badge in the explorer as soon as it authors blocks — that comes straight from the chain and needs nothing extra. But **Best Block / Fin. Gap / Version / Uptime** read `—` for a node-only validator, because those come from a heartbeat that a cert-daemon normally sends. If you want those columns filled, run the **lite-heartbeat** — a ~120-line Python client that reads those stats from your node's RPC and posts them signed with your aura key. It only reports liveness; it can't upload data or spend anything, and your key never leaves the box.

No sign-up needed — the gateway accepts heartbeats from any active committee member automatically (being seated on-chain is the enrollment; expect a 403 until your first committee selection).

```bash
pip install substrate-interface
cd /opt && sudo curl -fsSLO https://materios.fluxpointstudios.com/releases/materios-lite-heartbeat.py
curl -s https://materios.fluxpointstudios.com/releases/SHA256SUMS | grep lite-heartbeat.py | sha256sum -c   # -> OK

# test one beat (points at your node's keystore dir; nothing secret is typed or stored):
ONESHOT=1 AURA_KEYSTORE_DIR=/path/to/base-path/chains/<chain-id>/keystore \
  python3 materios-lite-heartbeat.py            # expect: ... -> OK {"status":"ok",...,"auth_tier":"sig-only"}
```

Within ~30s your explorer row fills in. To run it continuously, grab the systemd unit + env template alongside it:

```bash
sudo curl -fsSLo /usr/local/bin/materios-lite-heartbeat.py https://materios.fluxpointstudios.com/releases/materios-lite-heartbeat.py
sudo curl -fsSLo /etc/systemd/system/materios-lite-heartbeat.service https://materios.fluxpointstudios.com/releases/materios-lite-heartbeat.service
curl -fsSL https://materios.fluxpointstudios.com/releases/lite-heartbeat.env.example | sudo tee /etc/materios-lite-heartbeat.env >/dev/null   # then edit: set AURA_KEYSTORE_DIR + NODE_PROC + VALIDATOR_LABEL (your explorer display name)
sudo systemctl daemon-reload && sudo systemctl enable --now materios-lite-heartbeat
```

Set `NODE_PROC` in the env file to your node's process name (default `partner-chains-node`) so uptime reports correctly. These stats are display-only — your real liveness and finality are proven on-chain by the blocks you author, heartbeat or not.

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
| Node stalls below tip, peers loop on "Banned, disconnecting" | Your node is requesting a historical range the FPS nodes can't serve, and gets scored as a repeat-requester and banned. Reputation decays in ~69s, it reconnects, re-requests the same range, and is re-banned — so it idles at a fixed height with 0-1 peers. This is on our side, not yours, and no peer-list change fixes it | Restore the **current** snapshot per [step 7](#7-restore-the-latest-data-snapshot) — landing at tip skips the ranges that trigger it, and forward-sync from there is clean. Do **not** add `--reserved-only`: it removes your ability to route around a banned peer and turns a transient ban into permanent isolation. Background: [OPERATOR_KIT.md → Sync stuck at snapshot floor](https://github.com/Flux-Point-Studios/materios/blob/main/docs/OPERATOR_KIT.md#sync-stuck-at-snapshot-floor--peer-ban-loop) |
| `sqlx::query: slow statement ... elapsed=2.5s` in node logs + repeated peer drops | Postgres missing `idx_ma_tx_out_ident` or untuned | Re-run [step 1](#1-provision-postgres-for-cardano-db-sync) |
| `registration-signatures` errors on `mainchain-signing-key` | Didn't strip the `5820` CBOR prefix from cold.skey | `jq -r '.cborHex' cold.skey \| sed 's/^5820//'` |
| `smart-contracts register` fails with `UTxO already spent` | Your `$REGISTRATION_UTXO` was consumed between prep + submit | Pick a fresh UTXO; re-sign (same sidechain / SPO keys are fine) |
| `registration-status` or `ariadne-parameters` errors with `Exported method CandidateValidationApi_validate_registered_candidate_data is not found` | Both commands run the **genesis** runtime embedded in `chain-spec-v6-raw.json`, which doesn't export that API. The live runtime does export it, but these commands never load the live runtime — see the note in [step 6](#verify-the-registration-landed) | Nothing is wrong with your registration. Use the db-sync / Koios / explorer check in [step 6](#verify-the-registration-landed) |
| The [step 6](#verify-the-registration-landed) check returns zero rows minutes after submit | Your registration tx never made it onto Cardano | Look the txhash up on [preprod.cexplorer.io](https://preprod.cexplorer.io/). If it's absent, re-run [step 6](#6-submit-the-registration) with a fresh `$REGISTRATION_UTXO` |
| You registered before, the tx is on Cardano, but [step 6](#verify-the-registration-landed) now finds nothing | That registration UTxO has been spent — replaced by a later registration, or deregistered | Use check **A** (`unspent = f` proves this) or check **C**. Check **B** cannot tell you this: Koios `address_utxos` returns only unspent UTxOs, so a spent registration and a missing one both come back empty. Re-register with a fresh `$REGISTRATION_UTXO` if you didn't intend to replace it |
| Registration is unspent on L1 but you're not in the committee | Normal for the first ~2 epochs — registration is immediate, Ariadne seating is not | Confirm the ~10-day window in [step 9](#9-wait-for-the-stake-snapshot) has elapsed, then see the next row |
| Registered and past E+2 but never selected | Stake too low vs other pools, or the registered bucket is already filled | Grow your pool's stake or wait for variance; D-parameter is `(15, 1)` today |
| Validator at peers=0 on a real Linux host | Inbound TCP 30333 unreachable | Open 30333/tcp on your firewall + cloud security group; confirm with `nc -zv <your-public-ip> 30333` from another network |
| Finality gap > 10 in explorer authority-lag panel | Committee under-quorum or your node behind | Compare your `chain_getFinalizedHead` to the explorer's — if you're the lagger, restart; if the chain is the lagger, check Discord for an active incident |

Full sync diagnostics, RPC reference, and recovery playbooks: [OPERATOR_KIT.md](https://github.com/Flux-Point-Studios/materios/blob/main/docs/OPERATOR_KIT.md).

## Getting help

- Materios Discord: open a `#support-ticket`
- Chain explorer: [fluxpointstudios.com/materios/explorer](https://fluxpointstudios.com/materios/explorer)
- Issue tracker: [Flux-Point-Studios/materios-operator-kit](https://github.com/Flux-Point-Studios/materios-operator-kit/issues)
- Preprod faucet: [docs.cardano.org/cardano-testnets/tools/faucet](https://docs.cardano.org/cardano-testnets/tools/faucet)
