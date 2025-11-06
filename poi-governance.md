---
hidden: true
---

# PoI Governance

E**very $AGENT you control inside the Clarity ecosystem is counted 1-for-1**, regardless of _why_ it’s there:

| Source of tokens                                                 | How they’re picked up by the weight file                                                                                       |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| **Regular Clarity stake** (tokens you already deposited to vote) | The Clarity contract itself exposes each voter’s token balance; the snapshot script simply reads that column.                  |
| **New Inference-Pool bonds**                                     | The script also scans the ledger for every `Bond UTxO`, sums the $AGENT inside, and adds that figure to the same wallet’s row. |

The CSV row used for voting therefore looks like:

```
scssCopyEditwalletBech32, votingWeight
addr1...xyz,  (Clarity stake) + (Bonded AGENT)   → total weight
```

So:

* If you hold **12 000 $AGENT** in a Clarity voting stake **and** bond **80 000 $AGENT** to run a 4090 pool, your voting weight for the next epoch = **92 000**.
* A pure voter with no pool just keeps the original 1:1 weight from their staked tokens.
* A new operator who bonds but hasn’t staked inside Clarity yet still gets full weight from the bonded amount.

Nothing else about the DAO changes—the proposal types, quorum calculation, and treasury execution all work exactly as before; we’re only feeding Clarity a richer weight file each epoch.
