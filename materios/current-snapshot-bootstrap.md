---
description: One-page verified rocksdb restore for a returning Materios preprod validator — fetch the current snapshot, sha256-verify, restore, point at the live bootnode, and confirm you're in the canonical GRANDPA room.
---

# Current-Snapshot Bootstrap

This is the hand-off recipe for a returning external validator (Draupnir, TrueAiData, Runir, or any operator re-joining after downtime). It does one thing: get your node onto the **current** preprod state without genesis-replaying into a stall or a divergent fork.

> **Why you need this.** partner-chains-node v1.8.0 has a historical-sync bug — a fresh node genesis-replays and stalls at block 0 with `Inherent error: Candidates inherent required`. Worse, a node that replays from an old database can finalize into a **divergent GRANDPA room** that never re-merges. The cure for both is to restore the published rocksdb snapshot, then prove you landed in the canonical room.

## 1. Restore the current snapshot

The snapshot is published hourly and sha256-verified. Read the manifest, pull the tarball, verify, restore:

```bash
mkdir -p ~/materios-preprod/data/chains
cd ~/materios-preprod

# Pull the current pointer
MANIFEST=$(curl -fsSL https://materios.fluxpointstudios.com/operator-snapshots/preprod/latest.json)
TARBALL_URL=$(echo "$MANIFEST" | jq -r .url)
EXPECTED_SHA=$(echo "$MANIFEST" | jq -r .sha256)

echo "$MANIFEST" | jq '{url, sha256, size_bytes, head_block_at_publish, published_at}'

# Download + verify before touching your data dir
curl -fLo snapshot.tar.gz "$TARBALL_URL"
echo "${EXPECTED_SHA}  snapshot.tar.gz" | sha256sum -c -   # MUST print "snapshot.tar.gz: OK"

# Restore only on a clean checksum
tar xzf snapshot.tar.gz && rm snapshot.tar.gz
```

The tarball contains only `data/chains/materios_preprod_v6/db/` — no keystore, no peer-id keys. Your validator identity stays yours. If the `sha256sum -c -` line does not print `OK`, stop: the download is corrupt or the manifest moved under you. Re-fetch the manifest and retry.

> **Re-restoring over an existing DB:** if you already have a stalled or divergent database, wipe it first — `rm -rf ~/materios-preprod/data/chains/materios_preprod_v6/db` — then untar. Leaving the old `db/` in place can leave you on the wrong fork.

## 2. Point at the live bootnode

There is exactly one reliable public bootnode. Use the DNS form — it tracks the node's WAN IP so it survives IP changes:

```
/dns4/bootnode.materios.fluxpointstudios.com/tcp/30333/p2p/12D3KooWPueKoxRAirTTKH4Y2qQAsJDegWMjS4k89Z7izCbZKgkM
```

Pass it on the command line:

```bash
materios-node-spo \
  --chain ~/materios-preprod/chain-spec-v6-raw.json \
  --base-path ~/materios-preprod/data \
  --validator \
  --name <your-label> \
  --port 30333 \
  --rpc-port 9945 \
  --public-addr /ip4/<YOUR.PUBLIC.IP>/tcp/30333 \
  --bootnodes /dns4/bootnode.materios.fluxpointstudios.com/tcp/30333/p2p/12D3KooWPueKoxRAirTTKH4Y2qQAsJDegWMjS4k89Z7izCbZKgkM
```

Do not use any `/ip4/...` bootnode address you may have from an older runbook — those egresses are dead. `--public-addr` is required for SPOs so peers can dial you back.

## 3. Post-sync divergence self-check

Restoring the snapshot is necessary but not sufficient — you must confirm you finalized into the **canonical** room. Compare your node's block hash at the network's finalized height against the network's hash at the same height:

```bash
NET=$(curl -fsSL https://materios.fluxpointstudios.com/chain-info)
H=$(echo "$NET" | jq -r .finalized_block)    # network's finalized height
HNET=$(echo "$NET" | jq -r .finalized)        # network's finalized hash at H

# Ask YOUR node for its block hash at the network's finalized height
HLOCAL=$(curl -s -X POST http://127.0.0.1:9945 \
  -H 'content-type: application/json' \
  -d "{\"jsonrpc\":\"2.0\",\"id\":1,\"method\":\"chain_getBlockHash\",\"params\":[$H]}" | jq -r .result)

if [ "$HLOCAL" = "$HNET" ]; then
  echo "CANONICAL room ✓  (local hash at $H == network $HNET)"
elif [ "$HLOCAL" = "null" ]; then
  echo "Not caught up yet — local node hasn't reached $H. Wait and re-run."
else
  echo "DIVERGENT room ✗  local=$HLOCAL  network=$HNET  → re-restore from step 1 and restart"
fi
```

**How to read it:**

- **Match** → you're in the canonical room. Done.
- **`null`** → your node simply hasn't synced up to `H` yet (the network tip moves while you sync). Wait a minute and re-run; this is not a fork.
- **Different non-null hash** → you finalized a different block at the same height as the network. That is a real divergent fork: wipe `data/chains/materios_preprod_v6/db`, re-restore from [step 1](#1-restore-the-current-snapshot), and restart.

Always compare at the **network's** finalized height (`H` from `/chain-info`), never at your local tip — your local node legitimately lags the network by a few blocks while catching up, and comparing at your own tip would give a false "diverged" reading.

## What "good" looks like

After restore + start, within a couple of minutes:

```bash
curl -s -X POST http://127.0.0.1:9945 \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"system_health"}'
```

Expect `peers ≥ 1` and `isSyncing` flipping to `false` once you pass the snapshot floor (`head_block_at_publish` from the manifest) and reach the live tip. Then run the [step 3](#3-post-sync-divergence-self-check) self-check — a `CANONICAL room ✓` is your green light to start producing.

For the full onboarding flow (keys, Cardano registration, Ariadne selection), see [SPO Onboarding](spo-onboarding.md). For hardware and network requirements, see [Node Requirements](node-requirements.md).
