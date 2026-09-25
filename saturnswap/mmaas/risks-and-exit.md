---
description: >-
  What can go wrong, when the keeper closes your book, what SaturnSwap does not
  promise, and the two transactions that take your whole book back to your own wallet.
---

# Risks, limits, and how to leave

Read this before you fund anything. Market making is not a yield product. You are holding both
sides of a market with real inventory, and the on-chain guarantees in
[What MMaaS is](README.md#custody-what-we-can-and-cannot-do) are about **custody**, not about
**outcome**.

## The keeper can close your book

The keeper does more than reprice. In the cases below it closes your book on its own: it cancels
the order it quotes for you and sends everything in it to the payout address your ceremony names,
and SaturnSwap signs and pays the network fee. SaturnSwap can also close a book by hand, to the
same address. **In every case the funds go back to your payout address.** The validator backs
this: a keeper transaction can only move your value to your order address, your payout address or
the bounded ADA fee leg (see [Custody](README.md#custody-what-we-can-and-cannot-do)).

Three of the four cases are your own terms, the ones you signed in your consent statement (see
[What the statement says](setting-up-a-book.md#what-the-statement-says)).

| The keeper closes your book when | How it is measured |
|---|---|
| **It is worth more than your book value cap** | Each round the keeper can price your token: the ADA in your order plus your token valued at the keeper's mid, which moves at most 10% per round. A rise in your token's price counts, so a rise alone can close a book that no one has traded with. |
| **It loses more than your daily loss limit in a UTC day** | Against the book's opening value for the UTC day: its lowest complete valuation in the first 10 minutes after the keeper first values it completely that day, measured the same way as the cap. A fall inside those 10 minutes lowers the opening value instead of closing the book. The round that first values your book each UTC day records that value and does not quote, and neither does a newly funded book's first round. Your limit is your share of the opening value: at the default 5%, once the first 10 minutes have passed, a book that opened the day at 100 ADA is closed once it is worth less than 95 ADA. A fall in your token's price counts as a loss even with no trade. Your token is valued at the keeper's mid, which moves at most 10% per round, so after a sharper move the verdict trails the market by a round or more. A round in which the keeper cannot value your book completely, for example while a fill is not yet confirmed, neither sets the opening value nor closes the book. |
| **Your terms are outside the published bounds, or two of your statements conflict** | Checked each round against [the bounds on your terms](setting-up-a-book.md#the-bounds-on-your-terms). The keeper never quotes on terms nearer the bounds than the ones you signed. It returns your order once. Two different statements with the same *signed at* time conflict, and the keeper returns the order rather than choose between them. |
| **No usable price for 15 minutes** | 15 minutes in a row in which either feed (bending.ai or GeckoTerminal) gives no price, or the two disagree by more than 5%, for any reason, including an outage at either provider. Every book reads the same two feeds, so a 15-minute outage at one provider closes every book at once. |

A close is not a pause. Your inventory is back at your payout address, as whatever mix of ADA and
token the book held at that moment, and nothing is quoted until you fund again.

**A rise alone can close your book.** The page lets you fund at most five sixths of your cap, so a
book funded at the most allowed and held entirely in your token is closed by a rise of more than
20% in its price; a book holding some ADA needs a larger rise. That rise is not rare. For a token
as volatile as NIGHT has been, a measured daily standard deviation of 5.68%, we estimate the
chance that its price touches +20% at some point as about 6% within 3 days, 22% within 7 days,
39% within 14 days and 56% within 30 days. These are estimates from a random-walk model with no drift, not
measurements. When it happens, the book comes back to your payout address at its higher value.

**Two cases are about capacity, not your book's value.** The keeper quotes one order per book. Any
other order at your order address is sent back to your payout address, up to two a day for each
client, and any beyond that rests unquoted. One keeper also quotes at most five books, and
SaturnSwap's own NIGHT book holds one of the five: a book that arrives while five are enrolled has
its order returned to your payout address, and it can enrol again once a seat is free.

**Funding the same instance again the same UTC day.** After a daily-loss close, that instance stays
closed until 00:00 UTC: anything you fund there before then is sent straight back to your payout
address. After any other close, including your own exit, the daily loss limit still measures
against the value your book opened that UTC day with, so a smaller book funded at the same
instance that day can be closed again as soon as the keeper values it. In both cases, until the
return lands, the new order rests at the edge of your range. Wait until 00:00 UTC, or set up a new
instance.

A feed-outage close is also remembered: that instance is not quoted again, even after the feeds
recover, and anything you fund there later is sent straight back to your payout address. To be
quoted again, set up a new instance.

A close needs what a reprice needs: a running keeper, a synced node and our gas. During a
[pause that stops every book](#pauses-that-stop-every-book-at-once), a close waits too. Your own
exit, below, needs none of them.

## Leaving, in two transactions

Both are built and signed in your browser, by your wallet, from
[saturnswap.io/v3/mmaas](https://saturnswap.io/v3/mmaas). Connect the wallet that owns the vault
and your instances are found **from chain alone**, with no file to upload and nothing for us to look up.
The exit steps sit under **Getting your funds out**.

### 1. Close the order

One transaction spends the resting order, burns its three beacons, and pays everything to the
payout address your ceremony was built with. Every unit in the payout comes from the order; your
wallet supplies only the network fee.

Before your wallet ever opens, the builder refuses if:

- the connected wallet is not the ceremony's payout address. The recovery is authorised by that
  key and nothing else can sign it;
- the credential is not **registered** on chain. The spend is authorised by the credential
  appearing in the transaction's withdrawals map, and an unregistered credential cannot appear
  there at all;
- the planned payout goes anywhere other than your payout address;
- the assembled CBOR fails an independent re-read. The bytes are checked against the order, the
  beacons and your address by code that shares nothing with the planner that produced them.

If your vault holds several resting orders, this closes one and tells you how many remain. Press
it again.

### 2. Retire the credential and take the 2 ADA back

Registering the vault cost a 2 ADA Cardano stake deposit. Retiring it returns that deposit in
full.

**Order matters, and the page enforces it.** Retiring the credential while anything still rests
at the order address leaves that inventory fillable by takers and repriceable by nobody,
including you, because your own recovery path withdraws from that same credential. So the retire
step stays closed until the chain says, twice over, that there is nothing to strand:

- the order address holds **no UTxOs**, and
- the chain still reports the credential as **registered** (an unreadable answer is not a yes).

Both are re-read in the moment before you sign, not once at build time. If it does happen, it is
stuck rather than lost: registration is permissionless, so anyone can re-register the credential
for 2 ADA to unstick it. Only your own key can retire it.

### What it costs, measured on mainnet

These are real transactions. Follow them on an explorer rather than taking the numbers from us.

| Step | Transaction | Fee |
|---|---|---|
| Close the order | [`cc709973…`](https://cexplorer.io/tx/cc70997302c0bb08d06baac3bebbf487b4c4c5b141b716aa1282388100f9861c) | 0.588766 ADA |
| Retire and reclaim | [`43ee9766…`](https://cexplorer.io/tx/43ee97661fabcca07f58b0d295950a16f8b210936aecd2014ec698513daa0932) | 0.337673 ADA |

Total 0.926439 ADA, against 2 ADA returned.

**Who signed the close and the retire.** They ran on a pilot book SaturnSwap funded with its own
ADA, so SaturnSwap's wallet signed them as that book's owner. Open either one on cexplorer and look
at the witness set: it carries the book's owner key and not the ADAM bot key, which is the property
that matters. No outside client has exited on mainnet yet, so this shows that the owner key alone
can empty the address; it is not yet a record of a third-party client leaving.

A **funded** book carrying real inventory went the same way in
[`181bde67…`](https://cexplorer.io/tx/181bde674d8a3baceba666647cc979fdf6c97052b2b1935337631292b304e7eb)
(fee 0.591181 ADA, block 13,855,967). Decoded from the chain: the order address held
27.000000 ADA and 120,605,319 units of NIGHT plus its three beacons; the transaction produced
**two outputs, both to the owner's address**, carrying 27.000000 ADA and 120,605,319 NIGHT
(every unit preserved), all three beacons burned, and the whole thing authorised by a
zero-lovelace withdrawal from the vault's own credential.

## Leaving if SaturnSwap is gone

The browser path above needs our page to be up. `escape.sh` does not need us at all. It ships in
the public verification repo:

```bash
git clone https://github.com/Flux-Point-Studios/saturnswap-maker-verify
```

### What it actually does

It refuses to attach any script it has not rebuilt from a public source and hash-checked against
your ceremony:

| Script | Rebuilt from | Must hash to |
|---|---|---|
| your applied `maker_stake_bound` | the validator source in that repo, re-applied with your nine parameters | your ceremony's `applied_script_hash` |
| the cardano-swaps two-way spend validator | the public [cardano-swaps](https://github.com/fallen-icarus/cardano-swaps) blueprint at tag **`v2.0.0`** | `dapp_hash` |
| the two-way beacon policy | that same blueprint, applied with `dapp_hash` | `beacon_id` |

Any mismatch is fatal and nothing is submitted. Because every input is public and every script is
checked locally, you can point it at **any node you trust**. A hostile node can refuse you
service, but it can never redirect your funds.

You can check one link of that chain right now, without running anything of ours. **The tag
matters**: cardano-swaps has moved on since the deployment SaturnSwap books against, and the
current default branch produces a different hash.

```bash
git clone --depth 1 --branch v2.0.0 https://github.com/fallen-icarus/cardano-swaps
python3 -c "
import json, hashlib
v = next(x for x in json.load(open('cardano-swaps/aiken/plutus.json'))['validators']
         if x['title'] == 'two_way_swap.swap_script')
print(hashlib.blake2b(bytes([2]) + bytes.fromhex(v['compiledCode']), digest_size=28).hexdigest())
"
# 11928a3ac3b65edbf103ea6bb3362e39b879a36f02897df31c40917b
```

That is the `dapp_hash` listed in
[The five values SaturnSwap publishes](setting-up-a-book.md#the-five-values-saturnswap-publishes),
and baked into your ceremony. It is a
third party's open-source contract, not ours.

### Running it

```bash
cd saturnswap-maker-verify
./escape.sh \
  --params my-ceremony.params.json \
  --cardano-swaps-plutus ../cardano-swaps/aiken/plutus.json \
  --project . \
  --network mainnet \
  --dest <where the funds should land> \
  --fund-addr <the wallet paying the fee> \
  --signing-key <your.skey>
```

You need Python 3, [aiken](https://aiken-lang.org) v1.1.22, and a `cardano-cli` pointed at a
node. Two things worth knowing:

- **Signing is offline.** `--sign-cli` can be a separate, air-gapped binary; `cardano-cli
  transaction witness` needs no node and no network. `--build-only` writes the body and stops, so
  the key never has to touch the machine that talks to a node.
- **Recovery runs in rounds.** Three attached scripts plus a 16 KB transaction ceiling means a
  large book does not fit in one transaction. Each round measures itself against the limits your
  node reports and halves the batch if it does not fit. Rounds are independent, so an interrupted
  run resumes by running the command again.

It will **refuse** rather than guess if your vault has unclaimed staking rewards, or if it cannot
read that balance at all. See [Staking and governance](#staking-and-governance).

**Keep your ceremony params file.** See
[Checking it yourself](setting-up-a-book.md#checking-it-yourself). If you lose it all nine values
are recoverable from chain, but that is a reconstruction, and a reconstruction is work you do not
want to be doing on the day you need it.

## The honest risk list

SaturnSwap ran books of its own on mainnet, with its own ADA, to find the sharp edges with its own
money at risk. The measurements below that say "our own book" come from them.

### You are holding inventory, on both sides

A two-way order is a standing bid and a standing ask. Takers hit whichever one is better for
them, which is by definition whichever one is worse for you at that moment. **You should expect
to end up with more of the token and less ADA after a fall, and more ADA and less token after a
rise.** That is what market making is. Over a trending move it is a loss against simply having
held.

Nothing in the design offsets this. Our fee is a fee, not a hedge.

### The band bounds the price, not the outcome

Your two floors are enforced by the validator on every quote, so the keeper can never bid above
your ceiling or ask below your floor. That is a hard constraint: an out-of-band quote is a
transaction the chain refuses.

**It says nothing about where the band sits relative to the market.** The band bounds the spread;
it does not put a floor under your loss. You chose those two numbers, and a band chosen around
the wrong mid is a band the validator will faithfully enforce.

### ⚠️ A book nobody re-quotes is not idle: it is a live mispriced offer

This is the risk most people get wrong, so it is worth stating flatly.

A resting order keeps the datum it was funded with. If nothing reprices it, that datum stays
exactly where it was while the market moves away. **A stale book is a standing offer at a stale
price**. If it has never been repriced at all, that price is your ceremony floors, which are
*the worst quote you ever authorised*.

We have measured this on our own book. One sat for 89.8 hours quoting a bid about 5% above the
market, with 27 ADA behind it. Nothing took it, so it cost nothing. That was luck, not a
property of the design.

Do not read "idle" as "safe". Ask which side is on the wrong side of the market, and how much
inventory is behind it. The **"last worked"** figure on the re-entry panel is how you tell, and
you can compute it yourself from any indexer.

### One band per instance

Your nine parameters are a pure function: the applied script hash **is** the stake credential
**is** the order address. Changing any parameter, including either floor, produces a different
instance at a different address, with its own 2 ADA deposit. There is no in-place edit.

Moving to a new band is a single signed transaction that carries your whole book across, and the
old instance's deposit is reclaimable, so the net cost is roughly the fees. But it is a move, not
an edit, and it is yours to initiate.

## Staking and governance

The ADA in your vault is yours, and so is the stake on it. At registration you choose a stake
pool and a governance position (Abstain by default). **Leave the stake pool blank**: today a book
stops being repriced at its first reward payout, for the reason in
[What changes the moment you delegate to a pool](#what-changes-the-moment-you-delegate-to-a-pool).
Blank means the vault's ADA stakes nowhere and earns nothing.

Staking yield is excluded from the inventory validator's fee basis in every generation after the
earliest: the basis subtracts any withdrawn rewards before the rate is applied. The earliest
generation does not do this, and [Validator generations](validator-generations.md) explains how to
tell which one your book uses.

### Why the default is Abstain, and not nothing

Your vault is registered by one certificate that also records a governance choice, so the page
always sets one. Abstain is the default because it is the neutral choice: it takes no governance
position and hands voting power to nobody. No confidence and a named DRep are equally valid.

No DRep delegation at all would not lock your vault either. Every owner action on your vault,
the keeper's included, is authorised by a withdrawal from its credential. Conway refuses a
withdrawal from a reward account with no DRep delegation (`ConwayWdrlNotDelegatedToDRep`), but
only when a key controls that account. Your vault's credential is a script, so the check never
applies to it. On preprod, one vault credential,
[`3227e143…`](https://preprod.cexplorer.io/stake/stake_test17qez0c2rsvpvdqpswfeadhpwxfs7fdsvxex4addseaasz5g8z86ce),
was registered with a plain certificate that set no governance choice and has never delegated to
a DRep. Since then, 14 transactions under protocol version 11.0, where the check is active, have
been authorised by a withdrawal from it. One of them is
[`a210b667…`](https://preprod.cexplorer.io/tx/a210b66797bdadf59d5fa2d86b2db2af190e60fddceb2663a6ce911c0918f174).

### What changes the moment you delegate to a pool

⚠️ Once a pool pays you, your vault's reward account is non-zero, and Conway requires a withdrawal
to drain that account **exactly**. Measured against a real reward account holding 4,331,166,337
lovelace: a withdrawal one lovelace short was rejected outright, naming both figures. It is an
equality, not a ceiling.

On the browser side this is handled. Every owner action (close, retire, move house) measures
the balance from chain, states it, and **re-reads it in the moment before you sign**. If an epoch
boundary moved it in between, the page refuses, tells you the old and new figures, and asks you
to rebuild. Nothing is signed and nothing is touched. Retiring drains the account and deregisters
the credential in the same transaction, on one signature.

**The keeper does not handle it yet.** It plans every reprice as if your reward balance were zero,
so the reprice it plans has no output to pay a reward into. Before building, it reads the real
balance from the node, and once that balance is non-zero it refuses the reprice, every round.
From your first reward payout, **your book stops being repriced and rests at its last quote**,
which is the stale-book exposure above.

So, plainly: **leave the stake pool blank.** A vault delegated to a pool today stops being quoted
at its first reward payout. Your funds are not at risk and your own exit still works: the browser
path handles a non-zero balance, and `escape.sh` refuses until you claim it and tells you so. But
the service stops working for you.

## Operational limits

### If a price feed goes down, your pair is not quoted: it is not quoted wrongly

The keeper prices your pair from two independent public feeds, with a divergence breaker between
them, and refuses an operator-set constant mid on mainnet. See
[What your token needs](README.md#what-your-token-needs) for the rule and the reason. The short
version: **fewer than two healthy feeds means the pair is not quoted that round**, because
divergence is unverifiable and quoting into an unchecked number is worse than not quoting. If it
lasts 15 minutes, the keeper closes your book. Every book reads the same two feeds, so an outage at
either provider does that to every book at once; see
[The keeper can close your book](#the-keeper-can-close-your-book).

A token with no listing on either feed cannot currently be quoted at all.

### Pauses that stop every book at once

One keeper quotes every book, through one Cardano node, paying gas from one operator wallet.
Either of these stops every book at the same time, whatever your market is doing:

- **The node stalls.** The keeper reads the chain and builds every transaction through that single
  node, with no failover. When the node's tip falls more than 4 minutes behind the clock, the
  keeper refuses the whole round rather than quote from a stale chain.
- **Our gas runs out.** Every reprice and every keeper close is paid from that one operator
  wallet. When it cannot cover a transaction, the keeper skips it, and no book is repriced until
  we top the wallet up.

While either lasts, your book rests at its last quote, which is the stale-book exposure above, and
the closes in [The keeper can close your book](#the-keeper-can-close-your-book) wait too. There is
no alert to clients: the **"last worked"** figure is how you tell. A pause cannot cost you custody,
and your own exit needs neither: your wallet signs and pays for it, and `escape.sh` runs against
any node you choose.

A price-feed outage also reaches every book at once, because every book reads the same two feeds,
but it does not pause them: after 15 minutes it closes every book, and each instance stays closed.
See [The keeper can close your book](#the-keeper-can-close-your-book).

Your daily loss limit is your book's alone. One book tripping its limit closes that book and no
other.

### What you can actually see

Honestly: not much, and you should not rely on us to tell you.

- The page shows each of your instances with what it holds and **"last worked *N*h ago"**,
  computed from the age of the UTxO at your order address.
- That number comes from the chain, so you can compute it yourself from any indexer without
  asking us. The age of the UTxO at your order address is the whole signal.
- There is **no public health endpoint and no alerting to clients today.** Our own book-health
  check is internal and deliberately not public. It is a list of which books are not being
  repriced and exactly what they hold, which is an adverse-selection target list aimed at the
  people it exists to protect.

If you are running real size, watch your order address yourself.

## What you still have to trust

The validator bounds where your value can go. It does not bound everything, and what remains is
short but real.

- **Quote quality is off-chain conduct.** The chain proves the keeper cannot take your inventory.
  It does not prove the keeper quotes well. Judge that from the public book, from "last worked",
  and from your own dashboard at [saturnswap.io/v3/mmaas/book](https://saturnswap.io/v3/mmaas/book).
- **The bot key touches your inventory on every reprice.** Repricing is the service: each reprice
  spends your order and rebuilds it. The validator fixes where the value can land, not whether the
  bot key can move it.
- **The fee line in your statement bounds one fee.** Its `our fee` line is the inventory
  validator's bound on the ADA fee leg of a keeper transaction: at most that share of the ADA the
  transaction pays out to your wallet, and nothing when it closes your order. The service fee is
  drawn from your separate prepaid balance, which that bound does not protect; see
  [How service billing works](README.md#how-service-billing-works).
- **The five published values are ours to publish and yours to check.** The verifier takes
  `dapp_hash` and `beacon_id` as given. Check them against
  [The five values SaturnSwap publishes](setting-up-a-book.md#the-five-values-saturnswap-publishes)
  and on chain.
- **Your keys are your own claim.** The setup records your wallet's key and payout address. No tool
  can prove to anyone else that they are really yours.
- **The verifier you run must be the real one.** A doctored clone can pass its own checks. See
  [Check an existing address](validator-generations.md#check-an-existing-address).
- **No outside client has exited on mainnet yet.** See
  [What it costs, measured on mainnet](#what-it-costs-measured-on-mainnet).

## What SaturnSwap does not promise

- **No guaranteed fills.** Your order rests on a public order book. Takers arrive or they do not.
  Thin markets stay thin.
- **No guaranteed profit.** See the inventory risk above. A market maker can quote perfectly and
  still finish behind on a trending move.
- **No guarantee the keeper is running at any instant.** It is one service. It can be down,
  [paused for every book at once](#pauses-that-stop-every-book-at-once), or refusing your book
  for a reason that is correct. There is no uptime commitment here.
- **No guarantee the code is free of vulnerabilities.** The security evidence is validator unit
  and property tests recorded on September 9, 2026, and red-team runs by Claude, Codex and Kimi
  models: the validator's most recent red-team record is from September 12, 2026, and the most
  recent combined run to reach a verdict, on September 10, 2026, returned BLOCK. See
  [Security evidence](security-evidence.md).
- **Bounded control over trading inventory.** The current validator requires keeper actions to
  preserve assets within the permitted order, client payout and bounded ADA fee outputs.
  The prepaid fee channel is separate and its operator signing branch can spend that balance;
  the inventory fee bound does not protect it. See [What it costs](README.md#what-it-costs).
- **No ability to stop you leaving.** Your key alone cancels every order and pulls the funds
  back, unconditionally, with no notice to us and no cooperation from us. We cannot deregister
  your credential to pocket its deposit, and we cannot delegate your stake.

A client who reads this page and proceeds anyway is a client who will not be surprised. That is
the point of it.

Back to: [What MMaaS is](README.md) · [Setting up a book](setting-up-a-book.md)
