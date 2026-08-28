---
description: >-
  What can go wrong, what SaturnSwap does not promise, and the two transactions
  that take your whole book back to your own wallet.
---

# Risks, limits, and how to leave

Read this before you fund anything. Market making is not a yield product. You are holding both
sides of a market with real inventory, and the on-chain guarantees in
[What MMaaS is](README.md#custody-what-we-can-and-cannot-do) are about **custody**, not about
**outcome**.

## Leaving, in two transactions

Both are built and signed in your browser, by your wallet, from
[saturnswap.io/v3/mmaas](https://saturnswap.io/v3/mmaas). Connect the wallet that owns the vault
and your instances are found **from chain alone** — no file to upload, nothing for us to look up.
The exit steps sit under **Getting your funds out**.

### 1. Close the order

One transaction spends the resting order, burns its three beacons, and pays everything to the
payout address your ceremony was built with. Every unit in the payout comes from the order; your
wallet supplies only the network fee.

Before your wallet ever opens, the builder refuses if:

- the connected wallet is not the ceremony's payout address — the recovery is authorised by that
  key and nothing else can sign it;
- the credential is not **registered** on chain. The spend is authorised by the credential
  appearing in the transaction's withdrawals map, and an unregistered credential cannot appear
  there at all;
- the planned payout goes anywhere other than your payout address;
- the assembled CBOR fails an independent re-read — the bytes are checked against the order, the
  beacons and your address by code that shares nothing with the planner that produced them.

If your vault holds several resting orders, this closes one and tells you how many remain. Press
it again.

### 2. Retire the credential and take the 2 ADA back

Registering the vault cost a 2 ADA Cardano stake deposit. Retiring it returns that deposit in
full.

**Order matters, and the page enforces it.** Retiring the credential while anything still rests
at the order address leaves that inventory fillable by takers and repriceable by nobody —
including you, because your own recovery path withdraws from that same credential. So the retire
step stays closed until the chain says, twice over, that there is nothing to strand:

- the order address holds **no UTxOs**, and
- the chain still reports the credential as **registered** — an unreadable answer is not a yes.

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

A **funded** book carrying real inventory went the same way in
[`181bde67…`](https://cexplorer.io/tx/181bde674d8a3baceba666647cc979fdf6c97052b2b1935337631292b304e7eb)
(fee 0.591181 ADA, block 13,855,967). Decoded from the chain: the order address held
27.000000 ADA and 120,605,319 units of NIGHT plus its three beacons; the transaction produced
**two outputs, both to the owner's address**, carrying 27.000000 ADA and 120,605,319 NIGHT —
every unit preserved — all three beacons burned, and the whole thing authorised by a
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
checked locally, you can point it at **any node you trust** — a hostile node can refuse you
service, it can never redirect your funds.

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

That is the `dapp_hash` published under **Check us, don't trust us** on
[saturnswap.io/v3/mmaas](https://saturnswap.io/v3/mmaas), and baked into your ceremony. It is a
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

**Keep your ceremony params file** — see
[Checking it yourself](setting-up-a-book.md#checking-it-yourself). If you lose it all nine values
are recoverable from chain, but that is a reconstruction, and a reconstruction is work you do not
want to be doing on the day you need it.

## The honest risk list

### You are holding inventory, on both sides

A two-way order is a standing bid and a standing ask. Takers hit whichever one is better for
them, which is by definition whichever one is worse for you at that moment. **You should expect
to end up with more of the token and less ADA after a fall, and more ADA and less token after a
rise.** That is what market making is. Over a trending move it is a loss against simply having
held.

Nothing in the design offsets this. Our fee is a fee, not a hedge.

### The band bounds the price, not the outcome

Your two floors are enforced by the validator on every quote, so the keeper can never bid above
your ceiling or ask below your floor. That is a hard constraint — an out-of-band quote is a
transaction the chain refuses.

**It says nothing about where the band sits relative to the market.** The band bounds the spread;
it does not put a floor under your loss. You chose those two numbers, and a band chosen around
the wrong mid is a band the validator will faithfully enforce.

### ⚠️ A book nobody re-quotes is not idle — it is a live mispriced offer

This is the risk most people get wrong, so it is worth stating flatly.

A resting order keeps the datum it was funded with. If nothing reprices it, that datum stays
exactly where it was while the market moves away. **A stale book is a standing offer at a stale
price** — and if it has never been repriced at all, that price is your ceremony floors, which are
*the worst quote you ever authorised*.

We have measured this on our own book. One sat for 89.8 hours quoting a bid about 5% above the
market, with 27 ADA behind it. Nothing took it, so it cost nothing — that was luck, not a
property of the design.

Do not read "idle" as "safe". Ask which side is on the wrong side of the market, and how much
inventory is behind it. The **"last worked"** figure on the re-entry panel is how you tell, and
you can compute it yourself from any indexer.

### One band per instance

Your nine parameters are a pure function: the applied script hash **is** the stake credential
**is** the order address. Changing any parameter — including either floor — produces a different
instance at a different address, with its own 2 ADA deposit. There is no in-place edit.

Moving to a new band is a single signed transaction that carries your whole book across, and the
old instance's deposit is reclaimable, so the net cost is roughly the fees. But it is a move, not
an edit, and it is yours to initiate.

## Staking and governance

The ADA in your vault is yours, and so is the stake on it. At registration you choose a stake
pool (optional — blank means it stakes nowhere and earns nothing) and a governance position
(Abstain by default).

Staking yield is **excluded from our fee basis** by the validator itself: the basis subtracts any
withdrawn rewards before the rate is applied. You are charged for the position being run, not for
the yield your own stake produced.

### Why the default is Abstain, and not nothing

Proven on a real Cardano ledger: a credential with **no DRep delegation at all cannot withdraw
its rewards** — the node rejects the transaction with `ConwayWdrlNotDelegatedToDRep`. Every
action on your vault, including closing it and including your own recovery tooling, is authorised
by a withdrawal from that credential. A vault that cannot move its rewards cannot be repriced,
closed, or recovered.

Abstain takes no governance position and hands voting power to nobody, and it keeps that door
open. No confidence and a named DRep are equally valid. "None" is the one that can brick you.

### What changes the moment you delegate to a pool

⚠️ Once a pool pays you, your vault's reward account is non-zero, and Conway requires a withdrawal
to drain that account **exactly**. Measured against a real reward account holding 4,331,166,337
lovelace: a withdrawal one lovelace short was rejected outright, naming both figures. It is an
equality, not a ceiling.

On the browser side this is handled. Every owner action — close, retire, move house — measures
the balance from chain, states it, and **re-reads it in the moment before you sign**. If an epoch
boundary moved it in between, the page refuses, tells you the old and new figures, and asks you
to rebuild. Nothing is signed and nothing is touched. Retiring drains the account and deregisters
the credential in the same transaction, on one signature.

**But the keeper does not measure it.** Its reprice transaction carries a zero withdrawal for
your credential, because nothing in the keeper reads a reward balance. Once your first staking
reward lands, the node will reject every keeper action on your book — which means **your book
stops being repriced and rests at its last quote**, i.e. the stale-book exposure above.

So, plainly: **if you delegate your vault to a stake pool today, expect quoting to stop at your
first reward payout.** Your funds are not at risk and your own exit still works — the browser
path handles a non-zero balance, and `escape.sh` refuses until you claim it and tells you so. But
the service stops working for you. **Leave the pool blank until this is fixed**, or watch your
book's age closely.

## Operational limits

### If a price feed goes down, your pair is not quoted — it is not quoted wrongly

The keeper prices your pair from two independent public feeds, with a divergence breaker between
them, and refuses an operator-set constant mid on mainnet. See
[What your token needs](README.md#what-your-token-needs) for the rule and the reason. The short
version: **fewer than two healthy feeds means the pair is not quoted that round**, because
divergence is unverifiable and quoting into an unchecked number is worse than not quoting.

A token with no listing on either feed cannot currently be quoted at all.

### Quoting can pause for reasons that are not about your token

The keeper runs a drawdown rail that stops it quoting when equity falls past its limit for
several cycles in a row. **A pause is not necessarily about your market** — the rail is not fully
isolated per book today, and per-book isolation is open work rather than a shipped property.

A halt cannot cost you custody: on any keeper action, your value can only reach your order
address, your payout address, or the bounded ADA fee leg, and the chain enforces that. What it
does cost you is quoting — and a book that is not being worked is exposed, not parked.

### What you can actually see

Honestly: not much, and you should not rely on us to tell you.

- The page shows each of your instances with what it holds and **"last worked *N*h ago"**,
  computed from the age of the UTxO at your order address.
- That number comes from the chain, so you can compute it yourself from any indexer without
  asking us. The age of the UTxO at your order address is the whole signal.
- There is **no public health endpoint and no alerting to clients today.** Our own book-health
  check is internal and deliberately not public — it is a list of which books are not being
  repriced and exactly what they hold, which is an adverse-selection target list aimed at the
  people it exists to protect.

If you are running real size, watch your order address yourself.

## What SaturnSwap does not promise

- **No guaranteed fills.** Your order rests on a public order book. Takers arrive or they do not.
  Thin markets stay thin.
- **No guaranteed profit.** See the inventory risk above. A market maker can quote perfectly and
  still finish behind on a trending move.
- **No guarantee the keeper is running at any instant.** It is one service. It can be down,
  paused by its own risk rail, or refusing your book for a reason that is correct. There is no
  uptime commitment here.
- **No third-party audit.** The protocol is red-teamed in-house rather than certified by an
  outside firm. That is a deliberate choice and you should weigh it as one.
- **No control over your inventory.** This one is a guarantee, in the other direction: we cannot
  send your value to ourselves, to a third party, or to any address that is not yours. That is
  enforced by the validator on every keeper action, and by consensus rather than by our conduct.
- **No ability to stop you leaving.** Your key alone cancels every order and pulls the funds
  back, unconditionally, with no notice to us and no cooperation from us. We cannot deregister
  your credential to pocket its deposit, and we cannot delegate your stake.

A client who reads this page and proceeds anyway is a client who will not be surprised. That is
the point of it.

Back to: [What MMaaS is](README.md) · [Setting up a book](setting-up-a-book.md)
