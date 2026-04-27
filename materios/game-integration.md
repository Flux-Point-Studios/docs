---
description: How to add Materios chain-of-custody anti-cheat to your game
---

# Game Developer Integration Guide

Materios provides **on-chain proof that a game score is real**. When a player finishes a run, your game client uploads the telemetry blob and submits a receipt to the Materios chain. A decentralized committee of attestors independently verifies the blob — and if it passes, the receipt is certified and anchored to Cardano L1.

This gives you tamper-proof, publicly auditable leaderboards with zero backend trust.

> **Permissionless by default.** Submitting receipts to Materios does not require an API key — any sr25519 keypair can sign and submit. Games need MATRA tokens for TX fees (free from the faucet on testnet). Blob uploads also use sr25519 signature auth with no API key. The **Orynq managed service** (API key) is an optional offering for studios that want Flux Point Studios to manage the full pipeline on their behalf.

***

## How It Works

```
Game Run Ends
    │
    ▼
Upload blob to Materios Blob Gateway
    │ (JSON telemetry: score, duration, fields...)
    ▼
Submit receipt to Materios chain
    │ (links receipt_id to content_hash)
    ▼
Cert daemon network verifies blob
    │ (fetch blob → integrity check → schema validation → sign attestation)
    ▼
Receipt certified on-chain (threshold met)
    │
    ▼
Checkpoint anchored to Cardano L1
    │ (Merkle root of certified receipts → Cardano metadata TX)
    ▼
Score is provably real — anyone can verify
```

***

## Step 1: Define Your Schema

Every game defines a **validation schema** that tells the attestor network what fields to expect and what bounds are plausible. Schemas live in the [schema registry](https://github.com/Flux-Point-Studios/materios/blob/main/cert-daemon/schemas/registry.json).

### Schema Format

```json
{
  "my_game_v1": {
    "game": "My Game Name",
    "version": 1,
    "description": "Run-complete payload for My Game",
    "fields": {
      "score":  { "type": "int",   "min": 0, "max": 1000000 },
      "level":  { "type": "int",   "min": 1, "max": 100 },
      "dur":    { "type": "float", "min": 1.0 },
      "player": { "type": "string" }
    },
    "computed_checks": [
      {
        "name": "score_rate",
        "condition": "dur > 0",
        "expr": "score / dur",
        "max": 500.0,
        "error": "Score rate implausible: {value:.1f}/s > {max}/s"
      }
    ]
  }
}
```

### Field Types

| Type | Description | Example |
|------|------------|---------|
| `int` | Integer value | `score`, `level`, `combo` |
| `float` | Decimal value | `dur` (seconds), `dist` (meters) |
| `string` | Text value | `player` (address or ID) |
| `bool` | Boolean | `completed` |

### Bounds

Each field can have optional `min` and `max` bounds. Values outside bounds cause the attestation to fail (the receipt is **not** certified).

### Computed Checks

For more sophisticated validation, computed checks evaluate expressions against field values:

| Property | Description |
|----------|------------|
| `name` | Human-readable check name |
| `expr` | Arithmetic expression using field names (e.g., `score / dur`) |
| `condition` | Optional guard — skip check if condition is false (e.g., `dur > 0`) |
| `max` | Static maximum for the expression result |
| `max_expr` | Dynamic maximum computed from fields (e.g., `(level * 100) * 1.1`) |
| `error` | Error message template with `{value}` and `{max}` placeholders |

### Submitting Your Schema

Open a PR to the [materios](https://github.com/Flux-Point-Studios/materios) adding your schema to `cert-daemon/schemas/registry.json`, or contact the Materios team in [Discord](https://discord.gg/MfYUMnfrJM).

You will also need a version number assigned in the `schema_lookup` table:

```json
{
  "schema_lookup": {
    "1": "clay_monster_dash_v1",
    "2": "my_game_v1"
  }
}
```

***

## Step 2: Upload Blobs

When a game run ends, upload the telemetry as a JSON blob to the Materios Blob Gateway.

### Blob Format

Your blob must be valid JSON with at minimum:

```json
{
  "v": 2,
  "score": 4500,
  "level": 12,
  "dur": 45.3,
  "player": "5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY"
}
```

- `v` — **Required.** Schema version number (matches `schema_lookup` in the registry).
- All fields defined in your schema must be present.
- Additional fields are allowed and ignored by validation.

### Upload API

```bash
# 1. Upload the blob (sr25519 signature auth — no API key needed)
curl -X POST https://materios.fluxpointstudios.com/blobs/blobs \
  -H "Content-Type: application/json" \
  -H "X-Public-Key: YOUR_SR25519_PUBLIC_KEY" \
  -H "X-Signature: SR25519_SIGNATURE_OF_BODY" \
  -d '{
    "data": "{\"v\":2,\"score\":4500,\"level\":12,\"dur\":45.3,\"player\":\"5Grw...\"}",
    "content_type": "application/json"
  }'
```

Response:

```json
{
  "contentHash": "0xabcdef1234567890..."
}
```

The `contentHash` is a SHA-256 hash of the blob data. You need this for the on-chain receipt.

The blob gateway authenticates uploads via **sr25519 signature** — no API key is required. Sign the request body with your sr25519 keypair and include the public key and signature in headers.

> **Orynq managed service (optional):** Studios that prefer a managed pipeline can request an API key from the Materios team in [Discord](https://discord.gg/MfYUMnfrJM). The Orynq service handles blob uploads, receipt submission, and monitoring on your behalf.

***

## Step 3: Submit Receipt On-Chain

After uploading the blob, submit a receipt to the Materios chain linking the `receipt_id` to the `content_hash`.

### Using Polkadot.js SDK

```typescript
import { ApiPromise, WsProvider, Keyring } from "@polkadot/api";
import { blake2AsHex } from "@polkadot/util-crypto";

const api = await ApiPromise.create({
  provider: new WsProvider("wss://materios.fluxpointstudios.com/rpc"),
});

const keyring = new Keyring({ type: "sr25519" });
const signer = keyring.addFromUri("YOUR_STUDIO_MNEMONIC");

// Generate a unique receipt ID (or derive deterministically)
const receiptId = blake2AsHex(contentHash + Date.now().toString());

// Submit receipt (v2 with player signature for anti-cheat attribution)
await api.tx.orinqReceipts
  .submitReceiptV2(
    playerPubkey,     // Player's sr25519 public key (32 bytes)
    playerSignature,  // Player's signature over the telemetry (64 bytes)
    1,                // sig_type: 1 = sr25519
    receiptId,
    contentHash,
    baseRootSha256,
    null,             // zk_root_poseidon (optional)
    null,             // poseidon_params_hash (optional)
    baseManifestHash,
    safetyManifestHash,
    monitorConfigHash,
    attestationEvidenceHash,
    storageLocatorHash,
    schemaHash,
  )
  .signAndSend(signer);
```

### Hash Fields

Most hash fields can be set to a zeroed 32-byte array (`0x00...00`) during initial integration. The critical fields are:

| Field | Required | Description |
|-------|----------|-------------|
| `receipt_id` | Yes | Unique identifier for this game run |
| `content_hash` | Yes | SHA-256 of the blob (returned by the gateway) |
| `base_root_sha256` | Yes | Merkle root of the blob chunks |
| `schema_hash` | Yes | Hash of the schema used for validation |
| `player_pubkey` | v2 | Player's public key for anti-cheat attribution |
| `player_sig` | v2 | Player's signature proving they produced the telemetry |

***

## Step 4: Wait for Certification

After the receipt is on-chain, the attestor network automatically:

1. Detects the new receipt via chain polling
2. Fetches the blob from the gateway using the `content_hash`
3. Verifies chunk integrity (SHA-256)
4. Validates the blob against your registered schema (field presence, types, bounds, computed checks)
5. Signs an attestation and submits it on-chain
6. Once threshold attestations are met (currently 2), the receipt is **certified**

Certified receipts are periodically flushed into a checkpoint Merkle tree and anchored to Cardano L1 as a metadata transaction.

### Checking Receipt Status

```bash
# Via RPC
curl -s -X POST https://materios.fluxpointstudios.com/rpc \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "orinq_getReceipt",
    "params": ["0xYOUR_RECEIPT_ID"]
  }'
```

A certified receipt will have a non-zero `availability_cert_hash`.

***

## Step 5: Read Certified Scores

### On-Chain Query

Use the Materios explorer or RPC to query certified receipts:

```bash
# Get receipt count
curl -s -X POST https://materios.fluxpointstudios.com/rpc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"orinq_getReceiptCount","params":[]}'
```

### Cardano L1 Verification

Every checkpoint is anchored to Cardano as a metadata transaction. The checkpoint contains a Merkle root of all certified receipts in the batch. Anyone can:

1. Find the Cardano TX on a Cardano explorer
2. Extract the Materios checkpoint metadata
3. Verify the Merkle proof for any individual receipt
4. Confirm the receipt was certified by the Materios attestor network

This gives you **Cardano-grade finality** for game scores.

***

## Architecture Overview

```
┌──────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Game Client │────▶│  Blob Gateway    │     │  Materios Chain  │
│              │     │  (stores blobs)  │     │  (stores receipts│
│  Uploads blob│     └────────┬─────────┘     │   + attestations)│
│  Submits TX  │──────────────┼──────────────▶│                  │
└──────────────┘              │               └────────┬─────────┘
                              │                        │
                    ┌─────────▼──────────┐             │
                    │  Attestor Network  │◀────────────┘
                    │  (cert daemons)    │  polls for new receipts
                    │                    │
                    │  Fetches blob      │
                    │  Validates schema  │
                    │  Signs attestation │
                    └─────────┬──────────┘
                              │
                    ┌─────────▼──────────┐
                    │  Cardano L1        │
                    │  (anchor TXs)      │
                    │  Merkle checkpoint  │
                    └────────────────────┘
```

***

## Example: Clay Monster Dash (Reference)

Clay Monster Dash is the first game integrated with Materios. Its schema (v1) requires these fields:

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `v` | int | `1` | Schema version |
| `score` | int | `>= 0` | Final score |
| `dist` | float | `>= 0` | Distance in meters |
| `dur` | float | `>= 3.0` | Duration in seconds |
| `crystals` | int | `>= 0` | Crystals collected |
| `combo` | int | `0-10` | Max combo achieved |
| `near_miss` | int | `>= 0` | Near-miss events |
| `slides` | int | `>= 0` | Slide events |
| `diff` | int | `1-20` | Difficulty level |
| `peak_mult` | float | `1.0-10.0` | Peak score multiplier |
| `player` | string | — | Player identifier |

Plausibility checks:

- **Speed**: `dist / dur` must be < 30 m/s
- **Crystal density**: `crystals / dist` must be < 0.65/m
- **Event density**: `(near_miss + slides) / dist` must be < 0.25/m
- **Score cap**: Score must be below a computed theoretical maximum based on distance, crystals, events, and peak multiplier

[View the Clay Monster Dash schema](https://github.com/Flux-Point-Studios/materios/blob/main/cert-daemon/schemas/registry.json)

***

## FAQ

### What happens if validation fails?

The receipt stays on-chain but is **not certified**. It's visible but ignored by any leaderboard that requires certification. There's no slashing or penalty — the score just doesn't get the Materios seal of approval.

### Can I update my schema?

Yes. Create a new schema version (e.g., `my_game_v2` with `"version": 2`), update the `schema_lookup`, and have your game client send `"v": 2` in new blobs. Old blobs with `"v": 1` continue to validate against the v1 schema.

### What if a field isn't in my schema?

Extra fields in the blob are ignored. Only fields defined in the schema are validated. This means you can include debug data, metadata, or future fields without breaking validation.

### How many attestors need to verify?

The current threshold is **2 of 10** committee members. This means at least 2 independent cert daemons from the 10-member committee must verify and sign before a receipt is certified. In practice, certification takes ~12 seconds.

### What does it cost?

- **Blob storage**: Free during testnet
- **Receipt submission**: Costs MOTRA (fee token), currently waived (min_fee=0)
- **Attestation**: Free for game developers — attestors pay their own TX fees

### Can I run my own attestor?

Yes. Anyone can run a cert daemon and earn **tMATRA** (testnet tokens, no economic value) for every receipt they help certify on preprod. Real-MATRA economics start at mainnet — see [Mainnet Roadmap](mainnet-roadmap.md). See the [Attestor Guide](operator-guide.md#attestor-only) for the one-command setup.
