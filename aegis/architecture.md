---
description: The on-chain rules of the Sovereign Book. Offers, sales, positions, settlement, the price reading, the relay and the signers.
---

# How Aegis Works On-Chain

This page describes the generation running on Cardano preprod. Every rule below is taken from the
validators and off-chain code of that generation, which has had no independent audit yet. Prices are
scaled by 1,000,000 on chain.

## 1. Scripts

| Script | Purpose | Hash or policy on preprod |
|--------|---------|---------------------------|
| Book | Holds standing offers | [`83af823b…`](https://preprod.cexplorer.io/script/83af823bcb1999895f99c958510a011f26f02ed06ff17d8a17c16fab) |
| Position | Holds lane Heads and positions, and mints their markers | [`5a010322…`](https://preprod.cexplorer.io/script/5a0103220a367738ba0b180a465fe08756490b921aab4e0a0b46b5aa) |
| Lanes | One-shot policy of the 4 lane Heads | [`5a39f046…`](https://preprod.cexplorer.io/policy/5a39f04603f9308c787840eb0909404196ea1de77dccfe0bbfd37fd9) |
| Observer | Withdraw script that attests the genuine reading | [`db93795e…`](https://preprod.cexplorer.io/script/db93795e9baefe4fd47185d209d7d845025bb24d4bf14cd298c42e2e) |

A sale spends one Head and returns it unchanged, so lanes bound concurrency: two sales on one lane
race at the ledger, and the loser retries on the next output. The Head's Underwrite is the one
coordinator of a sale. A position's P-leg is the one coordinator of its settlement. Every other
spend proves only that its coordinator runs in the same transaction.

## 2. Offers

An offer is a UTxO at the book script, under the owner's own stake credential. It holds only
lovelace and carries this datum:

```aiken
type OfferDatum {
  owner:         VerificationKeyHash,
  manager:       Option<VerificationKeyHash>,  // the offer's agent
  approval:      Bool,                         // the agent witnesses every Fill
  feeds:         List<ByteArray>,
  classes:       Int,                          // bitmask: 1 Barrier, 2 Depeg
  sheet:         ByteArray,                    // blake2b_256 of the price sheet
  load_bps:      Int,                          // scales the sheet's rate
  min_depth_bps: Int,
  max_depth_bps: Int,
  min_term_ms:   Int,
  max_term_ms:   Int,
  min_fill:      Int,
  max_fill:      Int,
  expires_at:    Int,
}
```

| Action | Who | Rule |
|--------|-----|------|
| Fill | A sale | A Head is spent beside the offer. The Head's walk judges the offer. An approval-mode offer also needs its agent's signature. |
| Cancel | Owner | The owner's signature. |
| Reprice | Owner or agent | Re-parks the whole offer at its own address under terms that only narrow: same owner, agent, approval mode and sheet, load no lower, expiry no later, ranges and feeds no wider. |
| Withdraw | Agent | Returns the whole value to the owner's key under the offer's own stake credential, tagged with the offer's out-ref. |
| Withdraw | Quorum of 2 of 3 | The same return, short by at most 0.6 tADA to pay the batch's fee. The transaction may contain nothing except returns. |

The agent and the quorum can only send an offer home.

### Price sheets

A sheet is a packed table of 11-byte cells: a class tag (0 Barrier, 1 Depeg), then five big-endian
u16 values for minimum depth, maximum depth, minimum term in days, maximum term in days and rate in
basis points. It sits as the inline datum of a reference input, and an offer pins it by hash. The
leg's premium must satisfy:

```
premium × 100,000,000 ≥ coverage × rate × (10,000 + load_bps)
```

Preprod's reference sheets are test pricing and belong to no market maker. Their outputs sit at a
test account, and a script lock for them follows.

## 3. A sale

A sale's Underwrite requires all of the following.

**Position shape**
- One P-leg and between 1 and 20 C-legs, with C-leg coverage summing to the policy's coverage and
  leg premiums summing to its premium.
- The buyer named as `insured` signs.
- The payout address, if any, belongs to the insured.
- The transaction's validity span is at most 20 minutes, and the start time is at or after its
  upper bound.

**Price and strike**
- The feed and class are on the position script's compiled table.
- The reading is at most 10 minutes (Barrier) or 5 minutes (Depeg) old at the start time.
- The strike is under the reading's trailing low.
- Barrier: the strike is at least 15% under the lower of the spot and the trailing low.
- Depeg: the strike is between 50% and 95% of the peg, and the lower of the spot and the trailing low
  reads at least 99% of the peg. Those two rules together leave at least 4.04% of distance.

**Term and premium**
- The term is at least 7 days and at most 180 days. The reference sheets sell up to 45 days.
- The total premium is at least 2 tADA and at least 2% of coverage.
- Each leg covers at least 2 tADA and pays at least 1 tADA, at least 2% of its coverage, and its
  pro-rata share of the minimum premium. Each leg's premium stays under its coverage.

**Each offer spent**
- It holds only lovelace and no reference script.
- Its leg sits at the position script under the offer's own stake credential, with the offer's owner
  and out-ref in the leg datum.
- The fill is inside `min_fill` and `max_fill`.
- The feed, class, depth and term are inside the offer's limits and inside the named sheet cell.
- The premium meets the sheet's ask.
- The sale's upper bound is at or before the offer's expiry.
- Any remainder re-parks at the same address under the same datum.

**Fee**
- One output pays the fixed fee address at least 1% of the premium, with the position's digest as
  its inline datum.

An offer funds its leg with coverage less premium, and the buyer supplies the rest. A C-leg
therefore holds exactly its coverage, and the premium is already inside it.

## 4. Settlement

Every settlement spends the P-leg and every C-leg of one position, burns the whole marker set and
mints nothing else. The only withdrawal allowed beside it is the observer's. No other script input
can run in the transaction, so no foreign script can count a payout as its own.

The validators place no rule on the P-leg's own deposit of about 2.1 tADA, which the buyer funded at
sale. Whoever submits the settlement can take it as change.

### Claim

Anyone can submit a Claim. It needs no signature from the buyer.

- With a trailing low on the reading, the low is at or under the strike and was observed inside
  `[start, expiry]`.
- Without one, the spot is at or under the strike and was observed inside `[start, expiry]`.
- The reading is fresh for the transaction: the validity range sits between its observation and its
  `valid_until`, and that validity is at most 2 hours.
- The transaction's lower bound is at least 1 hour after the start.
- The payout address receives at least the full coverage.

The trailing low lets a fall late in the term stay claimable after expiry, for as long as a fresh
reading still reports it. Its window is 46 hours.

### Expire

Anyone can submit an Expire once the transaction's lower bound is past expiry plus a 48-hour claim
grace, which is the 46-hour low window plus the 2-hour reading validity. Each leg's full value
returns to its owner's key under the leg's stake credential. The premium is the underwriter's return.

### Buyer cancel

The insured signs within 30 minutes of the start. The reading used must have its spot above the
strike, and its trailing low, if it carries one, above the strike too. The buyer receives 90% of the premium, summed across legs. Each owner receives
the leg less its refund. The fee stays paid. The window closes before the 1-hour claim wait opens.

## 5. The price reading

A position settles only against its own feed, read through the observer's withdrawal. The observer
attests that the reading is genuine, and the freshness rules stay in the position script. This
generation pins the AegisSelf feed and sets no second feed on a position.

The Book's preprod table:

| Feed | Class | Policy |
|------|-------|--------|
| ADA/USD | Barrier | [`e9470982…`](https://preprod.cexplorer.io/policy/e9470982835ad4b077c615a3b32710a44a14b8f1af3e5ad106401362) |
| BTC/USD | Barrier | [`1b1a6de9…`](https://preprod.cexplorer.io/policy/1b1a6de944240a9f95e6cf80fe77266d91c7034a8e34009c112dc08e) |
| ETH/USD | Barrier | [`8ceb3301…`](https://preprod.cexplorer.io/policy/8ceb33011ce19fdc4e743a1bb9e5b42180ed706c263fcb0ea7fa0347) |
| USDC/USD | Depeg | [`7902d589…`](https://preprod.cexplorer.io/policy/7902d589349c3fa80c0868d3e908179072225e7fe3a58531b28e0f02) |
| RH1/EVT | Barrier, event slot | [`8ef4aca7…`](https://preprod.cexplorer.io/policy/8ef4aca72f671b5c81cc556594be31215be4ed284a90d021ae19b158) |

The publisher signs price bundles through its signing core. Before signing, the core's policy
requires:

- a quorum of at least two exchange sources, no older than 5 seconds;
- a datum whose price, time and 4,200-second validity match that reference;
- a trailing low it can find in the observer's history, where samples are Pyth-signed and anchored on
  chain under metadata label 8746.

The publisher does not yet serve RH1. The slot is on the table, and no event cover is on sale.

## 6. The relay

The relay is off-chain and allocates a sale across offers. It ranks admitted offers by price, then
by when they were posted, then by out-ref, and searches for the lowest all-in cost. Each body it
presents leaves the buyer at least 120 seconds to sign and carries a mark at metadata label 1986:
the allocation log entry's hash and the mark key's signature over that hash and the body.

The validators do not read the mark. It is evidence of which allocation produced a sale. The relay
stops allocating when a halt slot is set.

## 7. Signers

Hosted roles sign through two signer stacks, which hold the keys for the operations below. The
observer's anchor carrier is separate and pays from its own fee account, which is not one of these
keys.

| Core | Operation | What it signs |
|------|-----------|---------------|
| Publisher | `publisher.price_bundle` | Feed updates |
| Value | `relay.mark` | The relay's sale mark |
| Value | `keeper.g2.settle` | Claims and Expires |
| Value | `maker.agent` | An offer's Withdraw as its agent |
| Value | `maker.float` | The maker's fee input and posts |

Each stack runs three separate processes:

- The **core** holds keys and reaches chain providers.
- The **policy worker** has neither keys nor network, and it evaluates a per-operation policy.
- The **broker** holds no keys and takes authenticated requests.

The core authenticates each caller against its registered operations, reads original chain data from
two pinned providers, and signs only the payload it derived itself. It enforces per-transaction and
daily caps and keeps a hash-linked audit log.

This is a preprod setup. Both stacks currently share one host and custody is relaxed, so the first-use
key ceremony and hardware custody follow later. Which operations have landed on chain is on the
[proof page](preprod-proof.md#signer-operations).

## 8. Constants

| Constant | Value |
|----------|-------|
| Minimum term | 7 days |
| Maximum term | 180 days |
| Barrier minimum depth | 15% |
| Depeg strike band | 50% to 95% of peg |
| Depeg minimum spot | 99% of peg |
| Claim wait after start | 1 hour |
| Buyer cancel window | 30 minutes |
| Buyer cancel refund | 90% of premium |
| Trailing low window | 46 hours |
| Maximum reading validity | 2 hours |
| Claim grace before Expire | 48 hours |
| Maximum legs | 20 |
| Minimum leg premium | 1 tADA |
| Minimum premium | 2 tADA |
| Premium floor | 2% of coverage |
| Preprod fee | 1% of premium |

## Earlier generations

These are history and describe no part of the Book.

- **V4 to V7.** A single shared pool UTxO held underwriter capital, with transferable `aLP` tokens and
  a per-policy marker token. Policies could name one of several oracle providers.
- **V8.** A stable vault replaced the pool, with derived accounting and M-of-N registry governance
  behind a 72-hour timelock. Builders were non-custodial: users signed every action. This is the
  generation deployed on mainnet, where new sales have been paused since 2026-09-04 and liquidity
  providers can withdraw.

Source for those generations is in
[aegis-contracts](https://github.com/Flux-Point-Studios/aegis-contracts). That repository does not
yet carry the Book's validators.
