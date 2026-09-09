---
description: >-
  Step by step from a connected wallet to a funded, self-custodial two-sided
  book — what you sign, what it costs, and how to tell each step worked.
---

# Setting up a book

Everything below happens at [saturnswap.io/v3/mmaas](https://saturnswap.io/v3/mmaas) in your
browser. You sign with your own wallet. There is no form and no approval step — but one part of the
flow is still manual, and it is named in [What is not automatic yet](#what-is-not-automatic-yet).

Read [What MMaaS is](README.md) first if you have not. This page assumes you already understand
that your inventory rests at an address only your key can empty, and that the band you choose is
permanent for that instance.

## Before you start

**A mainnet CIP-30 wallet with a key-based address.** The page detects Eternl, Vespr, Lace, Nami,
Typhon, NuFi, Gero, Begin and Yoroi from `window.cardano`. Your address must be an ordinary
payment-key address — base or enterprise — because the page reads your escape-hatch key from its
payment credential. A script-controlled address yields no identity, and the flow will not start.

**Stay on one address for the whole flow.** Your address is one of the nine ceremony parameters.
The funding step refuses to build if the connected wallet is not the exact payout address your
ceremony bound, by name. Switching account or address index part-way through gives you a
different instance.

**The token, in that wallet.** Step 1 lists what the connected wallet actually holds. Nothing is
typed.

**Decimals and a live price** — see
[What your token needs](README.md#what-your-token-needs). Both are checked before you can build.

### ADA, and the shape it is in

| What | How much | Comes back? |
|---|---|---|
| Credential deposit | **2 ADA** | Yes, when you retire the credential |
| Registration fee | 0.337761 ADA on the measured mainnet certificate [`3c078a38…`](https://cexplorer.io/tx/3c078a38d28899423572db1635ca6ad924144bacce367189ca6f2c552f352a3d) | No |
| Order min-UTxO | **2 ADA**, resting in the order alongside your inventory | Yes, when you close the order |
| ADA you want quoted on the bid side | your choice | Yes, less whatever gets bought with it |
| Funding fee | 0.293094 ADA on the measured mainnet create [`dab1fd6a…`](https://cexplorer.io/tx/dab1fd6a38cea94d6b6176c08596e8a4e8ab3f0b62970ec052e1579777988466) | No |
| Collateral UTxO | **at least 7 ADA**, set aside rather than spent | Yes — it is never consumed |

The funding step also requires the inputs it picks to cover the order's own lovelace plus about
5 ADA of fee headroom. In practice: hold **roughly 20 ADA free** beyond whatever ADA you intend
to rest in the book, and expect about 0.6 ADA of it to be genuinely spent.

{% hint style="danger" %}
**The collateral footgun. Your wallet needs a pure-ADA UTxO of at least 7 ADA holding no native
tokens.** Not 5, and not 6.

The builders set 5 ADA of that UTxO aside as script collateral, and the remainder has to come
back as its own output, which must itself clear the minimum for an output. A UTxO of exactly
5 ADA leaves nothing to return, and the build fails **reporting a shortfall on the change
output** — which reads as "you need more ADA" and is not what is wrong. Sending more ADA does not
fix it: the wallet in the report that produced this rule held 890 ADA.

If you hit it, the page offers a **Set up collateral** button that carves a clean 10 ADA box off
what you already hold — a payment from your wallet back to itself, so you keep every lovelace —
and then re-runs the build. You can also do it by hand: send yourself about 10 ADA on its own,
then retry.
{% endhint %}

## 1 · Choose the token to quote

Connect your wallet from the header; the page shows a **Mainnet** pill next to it. Until you do,
step 1 reads: *"Connect the wallet holding the token. Everything else on this page is read from
it."*

Once connected you get a grid of the tokens that wallet holds. Pick one.

If this wallet has set up instances before, a **Pick up an instance you already set up** section
appears above step 1.

**Signs nothing. Costs nothing.**

## 2 · Choose the prices you will trade between

The page tells you three things, each a measurement rather than a default:

- what your token is trading at, and which feeds said so;
- how much it has moved per day on average, over a measured window;
- a suggested range, sized against that volatility rather than a constant.

You then edit two boxes: **Keep quoting while it is above** and **…and below**.

Underneath, the page restates what you have chosen: the range across which both sides quote, the
two protocol floors (*"Never buy above X, never sell below Y"*), and how long that range is
expected to keep both legs quoting at the measured volatility. Widening the range makes it last
longer but quotes further from the market. That trade is the whole decision, and it is the only
genuinely irreversible one you make.

If the market is already outside the range you typed, **Set up my instance** is disabled until
you widen it. If the range is wider than the spread can serve, you get an explicit *unreachable*
message naming the widest range that spread can reach.

Press **Set up my instance**. The nine parameters are assembled and the address is derived on the
server — never in your browser, because the browser build of the Cardano library applies script
parameters differently and would produce a different address from the same inputs.

**Signs nothing. Costs nothing.** Afterwards you see your order address, with the line *"Nothing
rests there until you fund it."*

**Editing either price after this builds a different instance, at a new address.** The page says
so and names the address you are walking away from. The old one keeps its own 2 ADA deposit and
anything resting there; the new one costs another deposit. Put the prices back to reach the old
instance again.

## 3 · Prove it's your wallet

One signature. No transaction, no fee, nothing submitted to the chain.

The message binds your wallet, your band and SaturnSwap's fee together, so nobody can show you
one ceremony and run another. Press **Sign with my wallet**, and the signed
`possession-proof.json` appears with a **Download** button.

**Your signature is published by you, in your own transaction.** It rides in the registration you
sign in the next step, as transaction metadata under label 8747. There is nothing to send us: no
Discord message, no email, no file transfer. The keeper reads it back off the chain, checks the
signature against the payout address your ceremony baked in, and checks that the challenge it
carries is the one your nine parameters hash to — so a proof for anyone else's ceremony, or for an
earlier version of yours, will not do.

**Take the download anyway.** It is your own copy of what you agreed to, and it is the input the
independent verifier needs to judge your ceremony offline.

**Sign this before you register.** The consent travels inside the registration transaction, so a
registration signed first carries none — and a certificate cannot be amended afterwards. The page
puts this step ahead of registering for that reason.

## 4 · Register it on chain

This is the transaction everything else waits on.

The panel first reads the chain to see whether this credential is already registered. If it
cannot read the chain it refuses rather than inviting you to pay a deposit blind, and re-checks
itself every 20 seconds, up to six times, before concluding this is an outage rather than indexer
lag and leaving the **Re-check** button to you.

You also choose, here and only here, what your own stake and governance weight do:

- **Stake pool** — optional. Leave it blank and the vault's ADA stakes nowhere and earns nothing,
  which is the default. SaturnSwap does not pre-fill its own pool. **Read
  [what changes the moment you delegate](risks-and-exit.md#what-changes-the-moment-you-delegate-to-a-pool)
  before you name one.**
- **Governance** — Abstain (default), No confidence, or a DRep you name. Abstain is the default
  for a mechanical reason, not a political one:
  [a credential with no DRep delegation cannot withdraw at all](risks-and-exit.md#why-the-default-is-abstain-and-not-nothing),
  and every owner action on your book is authorised by a withdrawal.

Press **Register my credential (2 ADA deposit)** and sign in your wallet.

**You sign:** one Conway `RegisterAndDelegateCredential` certificate, witnessed by your
instance's own Plutus script. The ledger accepts it only with your signature.

**It costs:** a 2 ADA deposit, refundable in full when you retire, plus the network fee —
0.337761 ADA on the measured mainnet certificate.

**How to tell it worked:** the panel flips to *"Your credential is registered on chain —
certificate `<hash>`"*, with a **View the certificate ↗** link and the line *"Funding can proceed
below."* If the indexers have not caught up you get the hash and the explorer link anyway, and
the panel keeps re-checking.

### If you are moving an existing book

When the page can tell you have replaced an earlier band, an extra step — **5 · Move your
existing order here** — appears between registration and funding, and funding renumbers to 6.
It moves your whole book from the old instance to the new one in a
single transaction you sign: no close and re-open, and your inventory never passes back through
your wallet. The move never makes your quote more aggressive than it already is — it carries the
resting quote across and raises only the leg the new band forbids.

## 5 · Put your inventory to work

The panel states the token you picked and how much of it you hold. Fill in:

- **Tokens to rest (ask side)** — in whole tokens; a **Use all** shortcut fills in your balance.
- **ADA to rest (bid side)** — optional. This is the ADA the keeper may buy with, inside your
  range. Leave it empty and your order can only sell until a sale gives it ADA to buy back with.

Press **Build the funding transaction**. Nothing is signed yet. What you get back is:

- a plain-English verdict — one output to your derived order address resting *N* token units with
  *M* lovelace, exactly three beacons minted, the fee, and every other output returning to your
  own wallet;
- a **Download** button for `body.tx`, the exact unsigned transaction body;
- the command to check it yourself with `verify_create_body.py` (see
  [Checking it yourself](#checking-it-yourself)).

Build and sign are separate on purpose: you can run the source-available Python gate over the downloaded
body before you sign, so signing never rests on this page's word.

Then press **Sign and submit with my wallet**.

**You sign:** one cardano-swaps two-way create — your inventory into your order address, three
beacons minted, an inline datum carrying both your bid and your ask.

**It costs:** the network fee (0.293094 ADA on the measured mainnet create `dab1fd6a…`), plus the
2 ADA min-UTxO that rests inside the order with your inventory and comes back when you close it.

**How to tell it worked:** *"Submitted: `<hash>`. Your order goes live once this lands…"*, with a
**View the transaction ↗** link.

The registration check runs twice — once before anything is planned, and again immediately before
the signature — so a credential retired in between cannot let a funding through.

## The order of operations that matters

**Register before you fund.** Every owner action on a cardano-swaps order is authorised by a
withdrawal from your credential, and an unregistered credential cannot appear in a withdrawal. An
order funded at an unregistered credential can still be **filled by takers** while being
**repriced or closed by nobody, you included** — until somebody registers the credential. It is
stuck rather than lost: registration is permissionless, so anyone can register it for the 2 ADA
deposit. The page enforces the order anyway: funding refuses unless the chain positively says the
credential is registered, and an unreadable chain is a refusal too, not a pass.

{% hint style="danger" %}
**Never send funds to your order address with an ordinary wallet transfer.** It is a script
address. A plain send arrives with no datum, and the validator decodes the datum before it
reaches any branch — so the transfer is permanently unspendable, by SaturnSwap, by you, and by
the wallet that owns it. The only safe funding is the transaction step 5 builds, which attaches
the beacons and the price datum. To add more later, run step 5 again.
{% endhint %}

**Close your orders before you retire the credential.** See
[Retire the credential](risks-and-exit.md#2-retire-the-credential-and-take-the-2-ada-back).

**A freshly funded order rests at the edge of your range** until the keeper's first reprice moves
it. That is the outermost price you authorised, and it is a gift to whoever trades against it —
it cost SaturnSwap 14.6 ADA on a live mainnet order, found deliberately on its own money. Fund a
small book first and let it be worked before you commit size.

## After funding

Once the create lands, your order is a live two-way order on SaturnSwap's book — anyone can fill
it at your posted prices, whether or not the keeper is working it yet.

What the keeper does once your book is enrolled: each round it reads a mid from the two feeds,
clamps the quote into your band, and rebuilds your order at the new prices. SaturnSwap signs and
pays the network fee for every one of those reprices.

### How to check your book is live

Come back to [saturnswap.io/v3/mmaas](https://saturnswap.io/v3/mmaas) and connect the same
wallet. Under **Already have an instance?** the page finds your instances from the chain — the
ceremony's own validator carries the numbers it was built with, so there is nothing to look up or
type — and lists each one roughly like this:

```
44.00 ADA · 0.0891 – 0.1094 ADA · last worked 3h ago · 3c265036cwm9…
```

**"last worked"** is the answer you want. It is the age of the resting UTxO, read from chain
alone, and it asks the keeper nothing: a refused ceremony, a missing config entry and a stopped
process are indistinguishable from outside, which is exactly why the check does not try to tell
them apart. A number that keeps climbing means nobody is repricing you.

You can compute the same number yourself from any indexer, or just watch your order address on a
block explorer. The per-client volume-and-fees dashboard at `/v3/mm` currently requires an API
key from SaturnSwap and is not self-serve.

## Checking it yourself

Two source-available Python tools ship in the public
[saturnswap-maker-verify](https://github.com/Flux-Point-Studios/saturnswap-maker-verify) repo,
and both work from a fresh clone with nothing from us:

- **`verify_create_body.py`** — judges the funding transaction body before you sign it. The
  funding panel prints the exact command and offers `body.tx`.
- **`verify_ceremony.py`** — rebuilds the validator from source, re-applies your nine parameters,
  derives the script hash and both addresses, and checks your possession proof against them.

Both take `--params my-ceremony.params.json`. **The guided flow does not hand you that file** —
only the possession proof. Two ways to get it:

- Open **Set this up by hand, from a cold key** on the same page and enter the two floors the
  guided flow showed you (*"Never buy above X, never sell below Y"*) plus your token's decimals.
  The expert form converts them with the same functions the guided flow does, so it reproduces
  the identical nine parameters and prints the file.
- Recover them from the chain: the applied validator carries all nine and rides in the witness
  set of your own registration transaction.

Keep that file. It is nine public values, and it is the simplest input to the recovery tool
described in [Leaving if SaturnSwap is gone](risks-and-exit.md#leaving-if-saturnswap-is-gone).

Cross-check the five values SaturnSwap publishes against a channel this page does not control.
They are listed under **Check us, don't trust us**, each with the chain query that confirms it.

## What is not automatic yet

Stated plainly, because acting on a stale claim here costs real money:

- **The keeper has to be pointed at your book once.** Your consent reaches the chain by itself, and
  the keeper verifies it there, but the address list it works is still set when the process starts.
  Your order rests and is fillable by takers in the meantime, at whatever price it last carried.
- **Enrolling a book in the keeper is still an operator step.** The keeper does not yet discover
  books from the chain by itself; the address list it works is set when the process starts. Your
  order rests and is fillable by takers in the meantime, at whatever price it last carried.
- **The guided flow does not give you your parameters file** — see above for the two ways to
  obtain it.
- **The client dashboard needs an API key.** Until then, "last worked" on the re-entry panel and
  a block explorer are the honest checks.

None of these can take your funds, and none of them can stop you leaving: every exit is
authorised by your signature alone.

Next: [Risks, limits, and how to leave](risks-and-exit.md)
