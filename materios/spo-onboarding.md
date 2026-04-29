---
description: End-to-end guide for Cardano stake pool operators registering as registered (SPO-weighted) candidates on the Materios preprod partner-chain
---

# SPO Onboarding (Preprod)

> **🧪 Preprod is a public testnet, not mainnet.** Rewards described here are paid in **tMATRA** — testnet tokens with no economic value. Running a preprod node is for testing the operator workflow and demonstrating uptime ahead of mainnet, not a revenue stream. See [Mainnet Roadmap](mainnet-roadmap.md) for the path to real-MATRA rewards.

This guide walks a Cardano stake pool operator (SPO) from zero to being a
**registered candidate** on Materios preprod — the role where your committee
selection probability is weighted by the stake delegated to your pool on
Cardano, rather than being in the permissioned set controlled by the Materios
governance multisig.

If you just want to run an attestor or permissionless full-validator, use
the [Operator Guide](operator-guide.md) instead. That path does not require a
Cardano stake pool.

> **Network:** Cardano preprod (magic `1`) ↔ Materios preprod.
> All ADA/tADA on preprod is worthless; keys generated below are testnet-only.
> Do **not** reuse a mainnet SPO cold key here.

***

## How Ariadne Selection Works (1-minute overview)

Materios' committee is chosen every Cardano epoch by **Ariadne**, IOG's
cross-chain committee selection algorithm. Each epoch the D-parameter decides
the split between two buckets:

- **Permissioned seats** — filled by the governance-controlled list (the 3
  Flux Point nodes today).
- **Registered seats** — filled by a random draw from candidates who
  submitted a valid registration on Cardano, weighted by the stake their pool
  had at the 2-epoch-prior snapshot.

Current preprod D-parameter (once you're eligible): **(3 permissioned, 2
registered)** after the upsert that accompanies your registration. Your
probability of being picked for a registered seat in epoch N is:

```
P(selected) = your_active_stake(snapshot @ epoch N-2) / total_registered_stake(snapshot @ epoch N-2)
```

Two consequences:

1. You must be registered on Cardano **at least one full Cardano epoch before**
   you can be selected. Registration in epoch E first counts for epoch E+2.
   On preprod (5-day epochs) that's a ~10-day wait.
2. Pledge is not the weight. Ariadne uses **active delegated stake**, which
   means you can self-delegate the pledged wallet's leftover tADA to inflate
   your weight on preprod.

***

## Prerequisites

| Item | Why | Where to get it |
|---|---|---|
| Linux host | Pool + Materios node | Any x86_64 Linux, 2 vCPU / 4 GB / 50 GB |
| Docker + Compose v2 | Running `cardano-node` and `materios-node` | docker.com |
| `cardano-cli` | Pool registration | Comes with `ghcr.io/intersectmbo/cardano-node:10.1.4` container (see below) |
| Cardano preprod node | Submitting txs | Option A: run your own. Option B: use public Ogmios + submit.saturnswap.io |
| IOG partner-chains-node | Registration signatures + on-chain registration | Pre-built binary: [v1.8.0 linux-x86_64](https://github.com/input-output-hk/partner-chains/releases/tag/v1.8.0) |
| ~1000 tADA | Pledge (100) + stake key deposit (2) + pool deposit (500) + fees (~3) + buffer | [Preprod faucet](https://docs.cardano.org/cardano-testnets/tools/faucet) |
| Materios chain spec | Connecting your node | `https://materios.fluxpointstudios.com/releases/chain-spec-v6-raw.json` |

### Option A — run your own preprod `cardano-node`

Reuse the reference compose stack from the [Operator Guide](operator-guide.md#prerequisite-cardano-preprod-stack).
You'll only need `cardano-node` itself for this guide (db-sync is not needed
for pool registration; it's only needed if you later run a Materios full
validator).

### Option B — public endpoints (limited)

Flux Point runs public Cardano infra for **mainnet only** right now
(`ogmios.saturnswap.io`, `kupo.saturnswap.io`, `submit.saturnswap.io`).
There is no public preprod endpoint today — Ogmios/Kupo on preprod are
LAN-only on Node-3. Pick Option A; request WAN access on Discord if you
need it for a short-lived demo.

> **Required for §5d:** the IOG `smart-contracts register` command speaks
> Ogmios. You need a reachable preprod Ogmios URL — either your own at
> `ws://<host>:1337` or via the temporary WAN tunnel if ops has one up.

***

## 1. Generate Keys

All cold operations are done **inside a disposable `cardano-node` container**
so you have the right `cardano-cli` version. Replace `~/cardano-preprod/keys`
with wherever you want the keys to live; that directory gets mounted into the
container.

```bash
mkdir -p ~/cardano-preprod/keys
cd ~/cardano-preprod/keys

# Drop into an ephemeral container that has cardano-cli
docker run --rm -it \
    -v "$PWD:/keys" \
    -v cardano-preprod-node-ipc:/ipc \
    -w /keys \
    --entrypoint bash \
    ghcr.io/intersectmbo/cardano-node:10.1.4
```

Everything from here to the end of this section runs **inside the container.**

### 1a. Cold / VRF / KES keys

> **Era prefix gotcha.** All key-gen commands below use the `conway`
> sub-group. Older tutorials use bare `cardano-cli node key-gen`, which errors
> on conway-era node versions. Always prefix with `conway`.

```bash
# Cold keys — operator identity. BACK THESE UP.
cardano-cli conway node key-gen \
    --cold-verification-key-file cold.vkey \
    --cold-signing-key-file cold.skey \
    --operational-certificate-issue-counter-file cold.counter

# VRF keys — block leader election
cardano-cli conway node key-gen-VRF \
    --verification-key-file vrf.vkey \
    --signing-key-file vrf.skey

# KES keys — block-signing (rotated every ~9 days)
cardano-cli conway node key-gen-KES \
    --verification-key-file kes.vkey \
    --signing-key-file kes.skey
```

### 1b. Payment + stake keys

```bash
# Payment keys (holds pledge, pays fees)
cardano-cli conway address key-gen \
    --verification-key-file payment.vkey \
    --signing-key-file payment.skey

# Stake keys (holds delegation, earns rewards)
#  ^ Note the era prefix: `conway stake-address key-gen`, NOT `stake-address key-gen`
cardano-cli conway stake-address key-gen \
    --verification-key-file stake.vkey \
    --signing-key-file stake.skey

# Build the stake-enabled payment address (this is what you fund + self-delegate)
cardano-cli conway address build \
    --payment-verification-key-file payment.vkey \
    --stake-verification-key-file stake.vkey \
    --testnet-magic 1 \
    --out-file payment.addr

# Plain stake address (used in certs, registration, pool metadata)
cardano-cli conway stake-address build \
    --stake-verification-key-file stake.vkey \
    --testnet-magic 1 \
    --out-file stake.addr

cat payment.addr; echo; cat stake.addr; echo
```

### 1c. Operational certificate (op-cert)

```bash
# Query current KES period
CURRENT_KES=$(cardano-cli conway query tip --testnet-magic 1 --socket-path /ipc/node.socket \
    | jq -r '.slot / 129600 | floor')
#         ^ slotsPerKESPeriod on preprod = 129600; check genesis if unsure

cardano-cli conway node issue-op-cert \
    --kes-verification-key-file kes.vkey \
    --cold-signing-key-file cold.skey \
    --operational-certificate-issue-counter-file cold.counter \
    --kes-period "$CURRENT_KES" \
    --out-file op.cert
```

**Back up everything in `~/cardano-preprod/keys/` immediately** (tar+gpg+offsite).
Pool keys are irreplaceable without re-registering the pool.

***

## 2. Fund the Payment Address

Exit the container for a moment to grab tADA:

1. Copy the contents of `payment.addr` (starts with `addr_test1...`).
2. Paste into the [Cardano preprod faucet](https://docs.cardano.org/cardano-testnets/tools/faucet)
   and request 10,000 tADA (one request is enough; you only need ~630 tADA to
   register with 100 pledge, but faucet drips come in 10k chunks).
3. Wait ~20 seconds for the tx to settle. Verify:

```bash
cardano-cli conway query utxo \
    --address $(cat payment.addr) \
    --testnet-magic 1 \
    --socket-path /ipc/node.socket
```

You should see one UTXO for ~10,000,000,000 lovelace.

***

## 3. Register the Stake Pool on Cardano

This is one transaction that bundles three certificates:

1. Stake-address registration (deposit: 2 tADA)
2. Pool registration (deposit: 500 tADA)
3. Self-delegation of your stake key to your pool

### 3a. Pool metadata

```bash
cat > pool-metadata.json <<'EOF'
{
    "name": "My Preprod Materios Pool",
    "description": "Cross-validates Materios partner-chain receipts",
    "ticker": "MPRE",
    "homepage": "https://example.com"
}
EOF

cardano-cli conway stake-pool metadata-hash \
    --pool-metadata-file pool-metadata.json > pool-metadata-hash.txt
cat pool-metadata-hash.txt
```

You need an HTTPS URL that serves this exact JSON file for the registration
cert. On preprod a placeholder URL is fine — the hash is what's verified
on-chain. For a real demo, host it at a stable URL.

### 3b. Generate certificates

```bash
# Stake address registration cert
cardano-cli conway stake-address registration-certificate \
    --key-reg-deposit-amt 2000000 \
    --stake-verification-key-file stake.vkey \
    --out-file stake-reg.cert

# Pool registration cert
cardano-cli conway stake-pool registration-certificate \
    --cold-verification-key-file cold.vkey \
    --vrf-verification-key-file vrf.vkey \
    --pool-pledge 100000000 \
    --pool-cost 170000000 \
    --pool-margin 0.01 \
    --pool-reward-account-verification-key-file stake.vkey \
    --pool-owner-stake-verification-key-file stake.vkey \
    --metadata-url "https://example.com/pool-metadata.json" \
    --metadata-hash $(cat pool-metadata-hash.txt) \
    --testnet-magic 1 \
    --out-file pool-reg.cert

# Self-delegation cert (points our stake key at our own pool)
cardano-cli conway stake-address stake-delegation-certificate \
    --stake-verification-key-file stake.vkey \
    --cold-verification-key-file cold.vkey \
    --out-file delegation.cert
```

### 3c. Build, sign, submit

```bash
# Grab a UTXO and set TIP
UTXO_IN=$(cardano-cli conway query utxo --address $(cat payment.addr) \
    --testnet-magic 1 --socket-path /ipc/node.socket --output-json \
    | jq -r 'to_entries[0].key')

cardano-cli conway transaction build \
    --tx-in "$UTXO_IN" \
    --change-address $(cat payment.addr) \
    --certificate-file stake-reg.cert \
    --certificate-file pool-reg.cert \
    --certificate-file delegation.cert \
    --witness-override 3 \
    --testnet-magic 1 \
    --socket-path /ipc/node.socket \
    --out-file tx.raw

cardano-cli conway transaction sign \
    --tx-body-file tx.raw \
    --signing-key-file payment.skey \
    --signing-key-file stake.skey \
    --signing-key-file cold.skey \
    --testnet-magic 1 \
    --out-file tx.signed

cardano-cli conway transaction submit \
    --tx-file tx.signed \
    --testnet-magic 1 \
    --socket-path /ipc/node.socket
```

Write down the TxID:

```bash
cardano-cli conway transaction txid --tx-file tx.signed
```

### 3d. Verify

```bash
# Pool ID in bech32
cardano-cli conway stake-pool id --cold-verification-key-file cold.vkey --output-format bech32
```

Search https://preprod.cardanoscan.io for that pool ID. Within ~2 minutes
you should see your pool listed as "Registered". This does **not** mean you
can be selected on Materios yet — see §5.

***

## 4. Generate Partner-Chain Sidechain Keys

Materios uses three kinds of Substrate keys per validator:

| Key | Type | Purpose |
|---|---|---|
| `sidechain` | `ecdsa` (secp256k1) | Partner-chain identity, used to sign registration |
| `aura` | `sr25519` | Block production |
| `grandpa` | `ed25519` | Finality |

The Flux Point validators use **mnemonic-derived** keys so any of the three
can be re-derived from a single 24-word phrase. Do the same:

### 4a. Generate mnemonic and derive all three keys

The IOG `wizards generate-keys` command is **interactive** — it takes no CLI
flags and prompts you for the base-path when run. It generates one 24-word
mnemonic, derives the ECDSA/sr25519/ed25519 keys from it, writes the Substrate
keystore files, and prints the three public keys.

```bash
mkdir -p ~/materios-spo
cd ~/materios-spo

/path/to/partner-chains-node-v1.8.0-x86_64-linux wizards generate-keys
# Follow prompts: it asks where to put keystore (answer: ./data) and prints keys.
```

When prompted, **also save the printed mnemonic to `~/materios-spo/.secret-mnemonic`
and back it up offline** — the wizard stores the mnemonic inside the keystore
files but does not write a separate copy. You'll want one for recovery.

The wizard prints three pubkeys:

```
Sidechain (ECDSA) : 0x02abcd…     <- used by partner-chains registration
Aura  (sr25519)   : 0x34ef…       <- block production
Grandpa (ed25519) : 0x56ab…       <- finality
```

### 4b. Prefer the manual path?

If you don't trust the wizard and want to do it by hand:

```bash
# 1. Mnemonic
/path/to/partner-chains-node-v1.8.0-x86_64-linux key generate \
    --scheme Sr25519 --words 24

# 2. Derive each pubkey — note the different --scheme per key
# Sidechain (ecdsa)
/path/to/partner-chains-node-v1.8.0-x86_64-linux key inspect \
    --scheme Ecdsa "<your 24 words>"
# Aura (sr25519)
/path/to/partner-chains-node-v1.8.0-x86_64-linux key inspect \
    --scheme Sr25519 "<your 24 words>"
# Grandpa (ed25519)
/path/to/partner-chains-node-v1.8.0-x86_64-linux key inspect \
    --scheme Ed25519 "<your 24 words>"
```

Record the public keys. You'll need them literally in §5.

### 4c. Extract the sidechain *private* key

For the `registration-signatures` step we need the raw ECDSA secret hex, not
the mnemonic:

```bash
/path/to/partner-chains-node-v1.8.0-x86_64-linux key inspect \
    --scheme Ecdsa --output-type json "<your 24 words>" \
    | jq -r '.secretSeed'   # 0x-prefixed 32-byte hex
```

Hold onto that hex as `SIDECHAIN_SKEY` for the next step. Treat it like a
private key — it is one.

***

## 5. Register on the Partner-Chain

This is a two-phase operation:

- **Phase A** — produce signatures over the registration message with both
  your Cardano cold key AND your sidechain ECDSA key.
- **Phase B** — submit a Cardano transaction to the CommitteeCandidateValidator
  address that includes those signatures as a datum.

Both phases use the IOG `partner-chains-node` binary. **None of this involves
the Materios node binary.**

### 5a. Pick a registration UTXO

The registration tx spends one UTXO from your payment address — any one you
control works, as long as it has at least a few tADA (~3 is plenty).

```bash
cardano-cli conway query utxo --address $(cat payment.addr) \
    --testnet-magic 1 --socket-path /ipc/node.socket --output-json \
    | jq -r 'to_entries[0].key'
# → e.g. 8b4e1f...#1
```

Record this as `REGISTRATION_UTXO`.

### 5b. Extract the 32-byte cold-key scalar

The `registration-signatures` command wants the **raw 32-byte CBOR payload** of
your cold.skey, not the whole `.skey` JSON file. Strip the `5820` CBOR prefix
from the `cborHex` field:

```bash
cat cold.skey
# { "type": "StakePoolSigningKey_ed25519", "cborHex": "5820abcd1234…" }

COLD_SKEY_RAW=$(jq -r '.cborHex' cold.skey | sed 's/^5820//')
echo "$COLD_SKEY_RAW"
```

### 5c. Phase A — generate signatures

```bash
GENESIS_UTXO=13313ea0119e0c4330f64f1809159064a371a1bbf2050b1fe13d5492280dca50#0

/path/to/partner-chains-node-v1.8.0-x86_64-linux registration-signatures \
    --genesis-utxo "$GENESIS_UTXO" \
    --mainchain-signing-key "$COLD_SKEY_RAW" \
    --sidechain-signing-key "$SIDECHAIN_SKEY" \
    --registration-utxo "$REGISTRATION_UTXO"
```

Output is a JSON object with `spo_public_key`, `spo_signature`,
`sidechain_public_key`, `sidechain_signature`, `registration_utxo`. Save the
whole thing; we pass most of it to the next command.

### 5d. Phase B — submit registration tx

> **Ogmios URL.** The IOG `smart-contracts register` command talks to Cardano
> via Ogmios, not `cardano-cli`. Pass `--ogmios-url` explicitly or set
> `OGMIOS_URL` in the environment. External SPOs should point at their own
> Ogmios running alongside their `cardano-node` (default port 1337).

```bash
/path/to/partner-chains-node-v1.8.0-x86_64-linux smart-contracts register \
    --genesis-utxo "$GENESIS_UTXO" \
    --registration-utxo "$REGISTRATION_UTXO" \
    --payment-key-file ~/cardano-preprod/keys/payment.skey \
    --partner-chain-public-keys "<SIDECHAIN_PUB>:<AURA_PUB>:<GRANDPA_PUB>" \
    --partner-chain-signature "<sidechain_signature from 5c>" \
    --spo-public-key "<spo_public_key from 5c>" \
    --spo-signature "<spo_signature from 5c>" \
    --ogmios-url ws://YOUR-OGMIOS:1337
```

The `--partner-chain-public-keys` format is literally
`SIDECHAIN_HEX:AURA_HEX:GRANDPA_HEX` with colons (no spaces, no 0x
repetition). Each value is the hex-with-0x-prefix pubkey you got in §4.

If the tx succeeds the CLI prints the submission TxID and polls for inclusion
(default 59 retries × 5s = ~5 min timeout).

### 5e. Sanity-check your registration

A couple of epochs later (but well before stake is effective):

```bash
/path/to/partner-chains-node-v1.8.0-x86_64-linux registration-status \
    --chain /path/to/materios-chain-spec-v6-raw.json \
    --stake-pool-pub-key "<SPO_PUB_KEY>" \
    --mc-epoch-number <current_cardano_epoch + 2>
```

This hits the Materios follower and reports whether the indexer has seen your
registration. If not, re-run with `+2` bumped until it does.

***

## 6. Wait for the Stake Snapshot

Cardano takes a stake snapshot at the **start of each epoch** and the one
used for Ariadne selection in epoch `E` is the snapshot that was frozen at
the start of epoch `E-2` (the `pparams_mark` → `pparams_set` → `pparams_go`
cadence).

Concretely, if you registered in Cardano epoch 283:

| Cardano epoch | Preprod date (5-day epochs) | What's happening |
|---|---|---|
| 283 | day of registration | Pool registered; stake visible in ledger but not yet snapshotted |
| 284 | +5 days | Snapshot taken (`mark`); Ariadne still doesn't count it |
| 285 | +10 days | Snapshot becomes `set` → **you're selection-eligible from here on** |

On top of that, the Materios team has to bump the D-parameter registered
bucket from `(3,0)` to `(3,2)` (or higher) so Ariadne even draws from the
registered pool. That happens in coordination with onboarding — reach out on
Discord once your pool is live.

During this wait you should:

- Start your Materios node (§8) so it's fully synced by the time you're
  eligible.
- Generate and back up your KES rotation plan — op-certs expire every ~9 days.
- Leave your Cardano pool alone; churning pool params resets the snapshot
  counter.

***

## 7. Verify Selection

Once the D-parameter shows registered ≥ 1 and at least one full epoch has
passed since your snapshot became `set`, check the live committee:

```bash
# Query the Materios chain via public RPC
curl -s -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"state_call","params":["SessionCommitteeManagementApi_get_current_committee","0x"]}' \
  https://materios.fluxpointstudios.com/preprod-rpc | jq .
```

Or via PolkadotJS Apps → **Developer › Chain State →
`sessionCommitteeManagement.currentCommittee`**. The authorities listed
should include your Aura pubkey for epochs where you were drawn.

Your expected draw rate per epoch:

```
P = your_active_stake / sum_of_active_stake_across_all_registered_candidates
```

On preprod with 2 registered seats and ~2 registered pools, you can estimate
how often your Aura key should appear. If you don't see your pubkey after
3–4 epochs in a row despite reasonable probability, open an issue.

***

## 8. Run the Materios Node (Full Validator)

You've already generated the sidechain/aura/grandpa keys (§4), so the
Materios operator-kit installer can be run in "bring your own keystore" mode.
Before running install, move the wizard-generated `keystore/` directory into
the location the installer expects:

```bash
mkdir -p ~/materios-operator
cp -r ~/materios-spo/data/chains/*/keystore ~/materios-operator/keystore
cp ~/materios-spo/.secret-mnemonic ~/materios-operator/.secret-mnemonic
```

Then run the installer with your existing mnemonic:

```bash
curl -sSL https://raw.githubusercontent.com/Flux-Point-Studios/materios-operator-kit/main/install.sh \
    | bash -s -- --label my-spo-validator
```

The installer detects the existing `.secret-mnemonic` and re-uses it (this
was hardened on 2026-04-16; earlier versions would overwrite). It wires up:

- **bootstrap-validator.sh** ([canonical install path](https://materios.fluxpointstudios.com/releases/bootstrap-validator.sh)) — fetches the v6 binary at [`/releases/materios-node-v6-x86_64-linux`](https://materios.fluxpointstudios.com/releases/materios-node-v6-x86_64-linux) (sha `dda4f3a7…58ebc`), the v6 chain-spec, and writes a systemd unit. The script verifies all SHAs against [`/releases/SHA256SUMS`](https://materios.fluxpointstudios.com/releases/SHA256SUMS) and refuses to run on unsupported environments (WSL2, Alpine, non-x86_64). The native binary needs glibc ≥ 2.38; pre-built arm64 / macOS binaries are not available — see [Operator Guide → Supported Environments](operator-guide.md#supported-environments) for the macOS manual path.
- **v6 data snapshot** at [`/releases/materios_preprod_v6-data-20260428-2245.tar.gz`](https://materios.fluxpointstudios.com/releases/materios_preprod_v6-data-20260428-2245.tar.gz) (sha `92641adf…7bd1`) — required to skip a known partner-chains-1.8.0 historical-sync bug. Apply after the bootstrap script finishes.
- chain spec from `https://materios.fluxpointstudios.com/releases/chain-spec-v6-raw.json`
- bootnode `/ip4/166.70.250.197/tcp/30333/p2p/12D3KooWPueKoxRAirTTKH4Y2qQAsJDegWMjS4k89Z7izCbZKgkM`
- port 30333 for P2P
- `cert-daemon` for attestation (also earns tMATRA on preprod — same testnet-no-economic-value caveat as block rewards)

> Runtime overrides are no longer needed on v6 — the operational patches (IDP-None fallback, Ariadne deduplication, GRANDPA queue-depth guard) are baked into the on-chain runtime (spec_version 211+).

### 8a. Recommended flags

The bootstrap script writes the systemd unit for you. If you build your own unit, the validator should be launched with:

```
materios-node-spo \
  --chain ~/materios-preprod/chain-spec-v6-raw.json \
  --base-path ~/materios-preprod/data \
  --validator \
  --name <your-pool-ticker> \
  --port 30333 \
  --rpc-port 9945 \
  --public-addr /ip4/YOUR.PUBLIC.IP/tcp/30333 \
  --bootnodes /ip4/166.70.250.197/tcp/30333/p2p/12D3KooWPueKoxRAirTTKH4Y2qQAsJDegWMjS4k89Z7izCbZKgkM
```

Setting `--public-addr` is **strongly recommended** for SPOs. Without it your
node advertises an internal IP to peers and other validators can't reach you
to gossip blocks you author — you'll produce blocks that nobody imports.

### 8b. Keystore format

Each key sits in `keystore/` as a **single file** named
`<key-type-4-bytes-hex><pubkey-hex>` (no `0x` in the filename). The key-type
prefixes are:

| Key | Prefix (hex of ASCII) | ASCII |
|---|---|---|
| Aura | `61757261` | `aura` |
| Grandpa | `6772616e` | `gran` |

Example filename for an Aura key with pubkey
`0xAAAA...AAAA` (32 bytes):

```
keystore/61757261AAAA...AAAA
```

File contents: a JSON-quoted string of **either** the 12/24-word mnemonic
**or** a `0x`-prefixed 32-byte seed. Substrate accepts both. The
`wizards generate-keys` command stores the mnemonic form (example shown
with the public Substrate dev mnemonic — NEVER commit your real one):

```
"bottom drive obey lake curtain smoke basket hold race lonely fit walk"
```

If you need to construct the keystore by hand (e.g. importing an existing
mnemonic), it's one file per key type, filename = prefix + pubkey-hex (no
0x), contents = `"<mnemonic>"` or `"0x<seed-hex>"` (quotes included).

### 8c. db-sync — do I need it?

Yes if you want to be a **full validator**. The Materios node queries Cardano
db-sync's Postgres directly to observe SPO registrations, D-parameter
changes, and epoch nonces. See the [Operator Guide](operator-guide.md#prerequisite-cardano-preprod-stack)
for the compose stack.

If you **only** want attestation rewards, you can skip db-sync and run
attestor mode — but there's no point doing the SPO registration in that case,
because nothing will read it.

***

## 9. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `registration-signatures` errors "invalid key length" | Passed full `cold.skey` JSON instead of 32-byte raw cbor hex | Re-extract: `jq -r '.cborHex' cold.skey \| sed 's/^5820//'` |
| `smart-contracts register` hangs on `Waiting for transaction` | Ogmios reachable but `cardano-node` not synced, or wrong magic | `cardano-cli conway query tip --testnet-magic 1` — ensure slot is within 60s of real time; check Ogmios logs |
| `registration-status` always returns `Pending` | Materios follower not yet caught up, or you're checking an epoch before your `set` snapshot | Wait 2 Cardano epochs after your registration tx, then retry |
| `sessionCommitteeManagement.currentCommittee` doesn't include you after 2 eligible epochs | D-parameter still `(3,0)` — no registered seats drawn | Check `MainChainScripts`/D-param upsert status with Flux Point ops |
| Chain spec `build-spec --raw` complains "unknown field `committeeCandidateAddress`" | Used camelCase inside an IOG sub-struct | Inside `mainChainScripts` the field is `committee_candidate_address` (snake_case). See [chain-spec serialization gotchas](https://github.com/...) |
| My Materios node panics at block proposal with "DParameter not found" | Chain spec stored an all-zero policy ID because an inner field was camelCased and silently Default'd | Same fix: snake_case for IOG sub-struct fields |
| `MainchainAddress` decoded wrong, follower can't find registrations | Bech32-decoded to 29-byte script header instead of hex-of-UTF-8-bytes | Re-encode: `python3 -c "print('0x' + 'addr_test1...'.encode('utf-8').hex())"` |
| I registered but have no MATRA so I can't pay for tx fees | You're running `materios-node` as a full validator; faucet drips trigger on first heartbeat | Wait ~60s after install completes; if still empty, ask in Discord |
| `KES period out of range` on node start | op-cert expired (they last ~9 days, 5 KES periods) | Re-issue: see [KES renewal](#kes-renewal-cadence) below |
| Node produces blocks but nobody imports them | `--public-addr` not set; peers can't reach you | Add `--public-addr=/ip4/YOUR.PUBLIC.IP/tcp/30333` to the compose command |

### KES renewal cadence

`op.cert` is valid for 5 KES periods (~9 days on preprod). When it expires:

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

Restart your `cardano-node` (not your Materios node) so it picks up the new
cert. The counter file is auto-incremented; do not edit it by hand.

### Getting Help

- Materios Discord: `#spo-support` channel
- Chain explorer: https://materios.fluxpointstudios.com
- Issue tracker: https://github.com/Flux-Point-Studios/materios-operator-kit/issues
- Cardano preprod faucet: https://docs.cardano.org/cardano-testnets/tools/faucet

***

## Appendix — Deployed Addresses (Preprod)

These are stable (genesis UTXO anchors their derivation). Hard-coding them in
your scripts is fine for preprod:

| Contract | Address |
|---|---|
| CommitteeCandidateValidator | `addr_test1wzr6en3y43437qps5wscegufxw0euspmy0c3976mjm95j0cwuvezm` |
| DParameterValidator | `addr_test1wqt6y9065lr0njwq2zq5yp6pjefuq5dc7nanvfnt2vz95vq80etl7` |
| PermissionedCandidatesValidator | `addr_test1wqne0zdkw4sj35x7sqxz3cqxqv7n7l5v853nxa7sw8uss9q7npmnm` |
| VersionOracleValidator | `addr_test1wz2zrp377qxkzlvjqejt8dwl6fqx4l6u4fh264n5xatptjq9y5uxu` |
| ReserveValidator | `addr_test1wpsr0vlpqjzfn3j5ggwdvyv49gwyuxp8mnjpnz096s28g0gdhgykd` |
| GovernedMapValidator | `addr_test1wrlzzlrd997kuz6pkqcly4al2l3y7e0ews445cthwhqap3qp4kq4a` |

**Genesis UTXO (v6):** `13313ea0119e0c4330f64f1809159064a371a1bbf2050b1fe13d5492280dca50#0`

**Partner-chain genesis hash (v6):** `0x0e46e33f639a56cc8780fd871d9a15e16d99af248526f907cb560cb40849f7bf`

**Committee candidate address (v6):** `addr_test1wrld9uhaepas48twjy3qevncsyrhjdqnkz2wzu4yzjc2qhq24f4v4`

Policy IDs (for reference — you shouldn't need these directly):

| Role | Policy ID |
|---|---|
| DParameter | `0x38dddaf5198b927b19dac9b28226ab29eddad176d5d81c7748bc2c31` |
| PermissionedCandidates | `0xef2890d1e98247819abcf2df6e891824ed950a4216d36c71ee6f9974` |
| VersionOracle | `0xc65dee78bf5208350433f62cf2d69e87c59cff5df86bf42d69ff7cbe` |
| ReserveAuth | `0xb4672fef1376b30ef2c010c97cbdb856701c9d776d1d989378a980f7` |
| GovernedMap | `0x6ee3f9e0812fe39ec82f299470636cfec42ff4887d359df6f83f2cf0` |
| ICS Auth | `0x29250512dd6b3244b06bfe88d7b808e5c5c8f076c4a92717ddffa7d7` |
