---
description: Cardano L1 Anchoring — Metadata format for Materios checkpoint transactions
---

# Cardano L1 Anchoring

## What Are Materios Anchor Transactions?

Every batch of certified receipts on the Materios chain is periodically checkpointed to Cardano mainnet as a **metadata transaction**. These anchor transactions carry a Merkle root covering all receipts in the batch, giving each receipt Cardano-grade finality without the cost of writing every receipt individually to L1.

- **Metadata label `8746`** is reserved for Materios receipt checkpoint anchors.
- The anchor worker submits a new checkpoint roughly **every ~2 minutes** when new receipts have been certified.
- Each anchor transaction costs approximately **0.18 ADA**.

## Metadata Format (v2)

All anchor transactions carry a JSON metadata payload under label `8746`:

```json
{
  "p": "materios",
  "v": 2,
  "chain": "5663079a485b93fdc9e386b862b4cf8d25499427df6b8c5f018535acfd2e5020",
  "root": "<merkle root of certified receipt batch>",
  "manifest": "<SHA-256 hash of batch manifest>",
  "blocks": [<from_block>, <to_block>],
  "leaves": <number of receipts in batch>
}
```

### Field Definitions

| Field | Type | Description |
|-------|------|-------------|
| `p` | string | Protocol identifier. Always `"materios"`. |
| `v` | int | Metadata schema version. Current: `2`. |
| `chain` | string | Materios chain genesis hash (first 64 hex chars). Identifies which chain this anchor belongs to. |
| `root` | string | Merkle root of the checkpoint batch. SHA-256 hash covering all certified receipts in this batch. |
| `manifest` | string | SHA-256 hash of the batch manifest JSON (contains leaf hashes, block range, timestamps). |
| `blocks` | \[int, int\] | Materios block range \[from, to\] covered by this checkpoint. |
| `leaves` | int | Number of certified receipts included in this batch. |

## How to Verify

1. **Find the TX on cexplorer.** Search for the transaction hash at [cexplorer.io](https://cexplorer.io). The metadata payload will be displayed under the transaction's metadata tab with label `8746`.

2. **Understand the Merkle structure.** Each leaf in the Merkle tree is computed as:
   ```
   SHA-256("materios-checkpoint-v1" || chain_genesis || receipt_id || cert_hash)
   ```
   The `root` field in the metadata is the root of this tree.

3. **Verify individual receipts.** Download the batch manifest (referenced by `manifest` hash). The manifest contains the ordered list of leaf hashes. Recompute the leaf hash for the receipt you want to verify and confirm it is present in the manifest; then recompute the Merkle root from all leaves and compare it to the on-chain `root`.

4. **Use the Materios Explorer.** The explorer at [materios.fluxpointstudios.com/explorer](https://materios.fluxpointstudios.com/explorer/) shows verification status for every checkpoint, including the corresponding Cardano TX hash and a direct link to cexplorer.

## Example Transaction

First mainnet anchor with v2 metadata, submitted April 10, 2026:

**TX hash:** `b2b55a984af345b3ae758d7c92ee033e3066b24c17b7b1755f07a30e9bababf3`

View it on cexplorer: [cexplorer.io/tx/b2b55a984af345b3ae758d7c92ee033e3066b24c17b7b1755f07a30e9bababf3](https://cexplorer.io/tx/b2b55a984af345b3ae758d7c92ee033e3066b24c17b7b1755f07a30e9bababf3)

## Anchor Frequency and Cost

| Parameter | Value |
|-----------|-------|
| Submission interval | ~2 minutes (when new certified receipts exist) |
| Cost per anchor TX | ~0.18 ADA |
| Metadata label | `8746` |
| Schema version | `2` |
