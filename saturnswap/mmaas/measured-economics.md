---
description: >-
  What market making usually costs a token project, what it cost on SaturnSwap's own measured
  book, and why the per-fill cost falls as more clients share the book.
---

# Measured economics

The measured figures on this page come from settled preprod transactions on SaturnSwap's own
book, its own taker filling its own maker. None of it is third-party flow, and the transaction
ladder below links to each transaction it cites. The desk comparison uses published DAO
governance figures and industry reporting, not chain data. Anything forward-looking is labelled
as planned. For what you pay and how billing works, see
[What it costs](README.md#what-it-costs).

## Compared with a market-making desk

Desk pricing is contractually private, so the only reliable figures are the ones token DAOs have
had to publish to pass a governance vote. Those show market-making retainers of **$3,500 to
$12,500 a month** for a single project, and industry reporting puts mandates that maintain active
quotes at **$15,000 to $50,000 a month**. The retainer is not the whole bill: the issuer also funds
the inventory the desk quotes with, commonly tens of thousands of dollars per venue.

| What you give up | Retainer model | Loan and call model | SaturnSwap MMaaS |
|---|---|---|---|
| Cash up front | $3.5k to $50k a month | None | A prepaid ADA fee balance; no retainer |
| Your tokens | You fund the inventory | 1 to 5% of circulating supply, for 12 to 24 months | None |
| Your upside | None | A call struck 25 to 100% above TGE | None |
| Ongoing | Fixed, whatever volume you get | The option is the fee | Default **0.20% of eligible settled ADA notional** |
| Custody | You fund a venue account | Tokens leave your treasury | Inventory in your own validator; fees in a separate channel |

**A retainer is fixed and this fee is not.** At the default 0.20% of eligible settled ADA notional,
a $12,500-a-month retainer is the cheaper deal only once you settle about **$75 million a year**.
Below that, a retainer is a six-figure annual fee for liquidity you may not be getting. The service fee accrues only on fills, so if the keeper quotes badly and nothing
fills, no fee accrues.

**The loan model has no cash fee, and its cost is harder to see.** You hand over a slice of supply
for a year or two and sell the desk an option on your own recovery. If the token runs, they
exercise. If it falls, they hand the tokens back. MMaaS never takes your tokens, so there is no
option to sell.

Retainer figures come from published DAO governance proposals and industry reporting. Desks do
not publish rate cards, and this is not a quote of any specific firm's price. Loan terms are the
ranges disclosed in DAO votes. The MMaaS rate is subject to your billing grant and any configured
cap; known operator fills are excluded, and an unfamiliar signer is labelled unattributed, not
independent. See [How service billing works](README.md#how-service-billing-works).

## The headline numbers

| | |
|---|---|
| Network gas, before any service fee | **0.0496%** of settled volume |
| All-in, with a 0.20% service fee applied | **0.2496%** |

On SaturnSwap's own book on preprod, with its own taker filling its own maker, **3,419,141 ADA** of
settled volume cost **1,696 ADA** in network fees. Applying a 0.20% service fee as an illustration
adds 6,838 ADA, for **8,535 ADA all-in**. For comparison, a conventional DEX charges **0.3% in
protocol fees alone**, before its own gas. SaturnSwap's earlier one-way approach cost 0.35%.

## Why more clients make it cheaper for everyone

Gas per fill depends on one variable: **how many fills share a transaction**. The token makes no
difference and neither does the owner; both were measured, below. So pooled order flow is the
pricing mechanism. Every client who joins deepens the batches every other client already rides,
and the cost per fill falls for all of them at once.

| Fills sharing one transaction | Gas per fill | Gas as % of volume | All-in with a 0.20% fee |
|---|---|---|---|
| 1, a maker running alone (n=260) | 0.4088 ADA | 0.213% | 0.413% |
| 4, a thin book (n=22) | 0.1785 ADA | 0.093% | 0.293% |
| **16, a pooled book (n=906)** | **0.0861 ADA** | **0.045%** | **0.245%** |

A maker running alone pays **0.413% all-in**. Inside a pooled book the same maker pays **0.245%**,
service fee included, which is still under what a conventional DEX charges in protocol fees before
any gas. The gas itself is **4.75 times lower** for identical volume.

The mechanism is a fixed floor. Every transaction of this kind pays roughly **0.29 ADA** in base fee,
Plutus bootstrap and reference-script overhead, however little it settles. A pooled transaction
pays that floor once for sixteen fills instead of sixteen times, and past the floor each additional
fill costs **0.063 ADA**, under a tenth of an ADA to settle roughly another 192 ADA of volume. At
scale, **5,000,000 ADA** of volume costs **2,244 ADA** pooled and **10,655 ADA** settled one fill at
a time.

Because per-fill gas does not depend on the token, and fills for different tokens batch into the
same transactions, the fee ratio holds near **0.04% at the margin** however many tokens the book
serves, while volume grows with the capital each client brings. If more clients join, volume
should grow with the capital they bring while the marginal cost ratio stays where it is; that is a
projection, not a measurement.

Percentages use the measured average fill size of about 192 ADA and the measured per-fill gas. The
0.20% service fee is an illustrative overlay at the default rate, not evidence of a paid invoice
from this operator-run rehearsal. Current billing excludes known operator fills. The overlay is
flat across the table, so movement in the all-in column comes from measured gas. Actual billing
follows each client's grant and eligible flow.

## Measured on our own multi-token book

The tables above come from that book: SaturnSwap's own maker, filled by its own taker agent, on
preprod. Across **17,823 settled fills in 1,405 transactions** quoting **five tokens** (ADAMMKT,
iUSD, RISE, AGENT, TEST), the keeper settled **3,419,141 ADA** of volume for **1,696 ADA** of
network fees: **0.0496%**, the gas figure in the headline.

Batches get deep by mixing tokens. A transaction carrying one token averaged 3.6 fills at
**0.210 ADA per fill**. Carrying three tokens, it averaged **16.0 fills** at **0.086 ADA per
fill**, **2.4 times cheaper** from mixing tokens alone. A fourth token held steady at 0.087 ADA per
fill.

| Token in the shared book | Fills | Volume | Gas per fill | Fee as % of volume |
|---|---|---|---|---|
| iUSD | 4,118 | 815,364 ADA | 0.0862 ADA | 0.0435% |
| RISE | 3,012 | 602,400 ADA | 0.0863 ADA | 0.0431% |
| AGENT | 403 | 80,580 ADA | 0.0867 ADA | 0.0434% |

Three tokens with very different volumes, and the cost per fill stays between **0.0862 and 0.0867
ADA**, within about 0.6%. **Gas belongs to the fill.** A client's cost does not depend on which asset they bring,
because their fills ride transactions the book was already paying for. At the measured rate that
is **4.75 times cheaper**, a saving of about **8,410 ADA** on 5,000,000 ADA of settled volume.

**Depth has a limit.** Sixteen fills is the measured optimum. The deepest transactions observed
(25 fills, a small sample) cost more per fill, as script execution units outgrew the amortisation.
The keeper tunes to the optimum.

## Evidence that pooling clients costs nothing

Everything above rests on the claim that putting several clients in one transaction costs nothing
extra, so it was settled on chain. Each client is its own stake credential, and their orders rest
at different script addresses, the same way two real clients' orders would. Every transaction
below is on preprod.

| Transaction | Fills | Distinct clients | Gas | Gas per fill | Against one fill per transaction |
|---|---|---|---|---|---|
| [`288730cd…`](https://preprod.cexplorer.io/tx/288730cd9c2aa278277422ee6c7ea9d037013ae178a1b0a21e11e143aba7f5ad) | 1 | 1 | 0.3949 ADA | 0.3949 ADA | 1.00x |
| [`a03177a5…`](https://preprod.cexplorer.io/tx/a03177a5a8bd6a481418cd3b0405057f4e89982d7941fe3f08807cb3604c1bc2) | 2 | 2 | 0.5183 ADA | 0.2592 ADA | 1.52x |
| [`eb66f7a7…`](https://preprod.cexplorer.io/tx/eb66f7a71095d5bb30f623a85412113108750593493f2462fafd2806d59356c9) | 4 | 2 | 0.8017 ADA | 0.2004 ADA | 1.97x |
| [`ab24ebff…`](https://preprod.cexplorer.io/tx/ab24ebff10f62da3b232b6a3baab99cc7842aff13a96acf6430bcdf45ec51378) | 8 | 2 | 1.3096 ADA | 0.1637 ADA | 2.41x |
| [`7a58d0fc…`](https://preprod.cexplorer.io/tx/7a58d0fc1220dfbaf33945d2e0b57b8cd75cae96bf07741b5c10ec7be00c492a) | 16 | 2 | 2.5695 ADA | 0.1606 ADA | 2.46x |
| **[`63fcd2f2…`](https://preprod.cexplorer.io/tx/63fcd2f2d7a7b9e067b0eacd5f2cea530cdbb252d768d658d27a8774553decfd) (control)** | **16** | **1** | **2.5695 ADA** | **0.1606 ADA** | **2.46x** |

Sixteen fills across two clients settled for **2.5695 ADA** in one transaction. As sixteen
separate transactions they would have cost **6.3184 ADA**, so the shared transaction was **2.46
times cheaper**. Every figure comes from the same wallet on the same pair, so the comparison hides
no other difference.

The last row is the control: the identical transaction with all sixteen fills under a single
client. It cost **exactly the same, 2.5695 ADA, at an identical 9,834 bytes**. An order address
differs between clients only in its stake credential, which has the same length either way, so the
fee cannot move. **Crossing clients costs nothing.**

Billing holds at the same depth. SaturnSwap's indexer read that sixteen-fill transaction back from
chain data and attributed **eight fills and 41.3466 ADA to each client**. One transaction becomes
two invoices with no manual reconciliation, which is what makes per-client billing on shared
transactions workable.

Read this ladder for its ratios. The test wallet holds a long tail of native assets that every
change output has to carry, which lifts its per-fill cost above the 0.086 ADA the multi-token
preprod book averaged. The control isolates the one variable under test, one client against two, and that
difference measured zero.

## Agents on both sides of the book

Resting orders only become volume when something fills them, so SaturnSwap runs the taker side
too: an autonomous agent that reads the live book, decides what is worth taking, and composes the
multi-fill transactions the economics above depend on. On the maker side, the keeper reprices
each client's order around the live mid as the market moves.

**Appetite is four numbers.** Every book set up on the page gets the same defaults:

| Setting | Value | What it does |
|---|---|---|
| Spread | 8% | The ask sits 4% above the live mid and the bid 4% below it, each clamped into the client's band. |
| Minimum reprice move | 0.5% | A quote is rebuilt only once the mid has moved at least this far from the one it is centred on. |
| Value cap | 120 ADA | A book worth more than this, its token valued at the live mid, is closed back to the payout address. |
| Daily loss | 5 ADA | A book whose value falls by more than this within a UTC day is closed back to the payout address. |

A client cannot change them on the page; SaturnSwap can set different values for a book by hand.
When and how a book is closed is in
[The keeper can close your book](risks-and-exit.md#the-keeper-can-close-your-book).

**Autonomy is bounded in two independent places.** The agent enforces its own risk configuration.
Beneath it, the validator bounds what any keeper-signed action can do: value can land only at your
order address, your payout address, or the bounded ADA fee leg (see
[Custody](README.md#custody-what-we-can-and-cannot-do)). Every agent runs on paper before it is
allowed to sign, and a kill switch halts it without touching client funds.

**Status.** The taker agent is built and rehearsed end to end on preprod (paper, live and kill),
including a real on-chain fill,
[`aea3d325…`](https://preprod.cexplorer.io/tx/aea3d325537d7afdcdb31803088304e681404951430abc08e72970883e459edd).
It ships behind an operator control with two-layer authentication. Running it on mainnet depends on
the onboarding of the client it would serve and the mandate it would run under. A preprod rehearsal
is not a mainnet record.

## Limits of this evidence

- **It is our own flow.** The preprod volume is SaturnSwap's own two-sided liquidity on a public
  order book, its maker filled by its taker. It measures gas, batching and billing, not demand.
  Planned: publish mainnet fills as they settle, each attributed to who took it. Public
  transactions support verification; they do not establish independent demand.
- **Token-to-token pairs earn no service fee today.** The meter values a fill by the ADA that
  moved, and a token-for-token fill moves none. What is missing is an ADA-equivalent notional for
  the fill. Planned: value such a fill at the keeper's price-feed rate for the settling slot,
  record that rate on the fill so you can re-derive and dispute the number, and bill it from the
  prepaid channel exactly as an ADA fill is billed. The gap is in measurement only, because the
  service fee is already drawn from the prepaid channel, not from the swap. The billing rail, the
  cap and the finality gate would carry over unchanged.
- **Batch depth is bounded by transaction execution limits.** On the measured preprod book the
  practical ceiling is about 16 fills; past it the cost per fill rises.
- **Security scope.** This page measures cost, not safety. For what has been tested, at which
  revision, and what that evidence does not cover, see [Security evidence](security-evidence.md).
- **The fee channel has different protections from your inventory.** See
  [How service billing works](README.md#how-service-billing-works).

Back to: [What MMaaS is](README.md) · [Risks, limits, and how to leave](risks-and-exit.md)
