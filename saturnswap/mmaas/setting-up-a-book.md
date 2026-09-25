---
description: >-
  Step by step from a connected wallet to a funded, self-custodial two-sided
  book: what you sign, what it costs, and how to tell each step worked.
---

# Setting up a book

Everything below happens at [saturnswap.io/v3/mmaas](https://saturnswap.io/v3/mmaas) in your
browser. You sign with your own wallet. There is no form and no approval step, but one part of the
flow is still manual, and it is named in [What is not automatic yet](#what-is-not-automatic-yet).

Read [What MMaaS is](README.md) first if you have not. This page assumes you already understand
that your inventory rests at an address only your key can empty, and that the band you choose is
permanent for that instance.

## Before you start

**A mainnet CIP-30 wallet with a key-based address.** The page detects Eternl, Vespr, Lace, Nami,
Typhon, NuFi, Gero, Begin and Yoroi from `window.cardano`. Your address must be an ordinary
payment-key address (base or enterprise), because the page reads your escape-hatch key from its
payment credential. A script-controlled address yields no identity, and the flow will not start.

**Stay on one address for the whole flow.** Your address is one of the nine ceremony parameters.
The funding step refuses to build if the connected wallet is not the exact payout address your
ceremony bound, by name. Switching account or address index part-way through gives you a
different instance.

**The token, in that wallet.** Step 1 lists what the connected wallet actually holds. Nothing is
typed.

**Decimals and a live price.** See
[What your token needs](README.md#what-your-token-needs). Both are checked before you can build.

### ADA, and the shape it is in

| What | How much | Comes back? |
|---|---|---|
| Credential deposit | **2 ADA** | Yes, when you retire the credential |
| Registration fee | 0.337761 ADA on the measured mainnet certificate [`3c078a38…`](https://cexplorer.io/tx/3c078a38d28899423572db1635ca6ad924144bacce367189ca6f2c552f352a3d) | No |
| Order min-UTxO | **2 ADA**, resting in the order alongside your inventory | Yes, when you close the order |
| ADA you want quoted on the bid side | your choice | Yes, less whatever gets bought with it |
| Funding fee | 0.293094 ADA on the measured mainnet create [`dab1fd6a…`](https://cexplorer.io/tx/dab1fd6a38cea94d6b6176c08596e8a4e8ab3f0b62970ec052e1579777988466) | No |
| Collateral UTxO | **at least 7 ADA**, set aside rather than spent | Yes, it is never consumed |

The funding step also requires the inputs it picks to cover the order's own lovelace plus about
5 ADA of fee headroom. In practice: hold **roughly 20 ADA free** beyond whatever ADA you intend
to rest in the book, and expect about 0.6 ADA of it to be genuinely spent.

{% hint style="danger" %}
**The collateral footgun. Your wallet needs a pure-ADA UTxO of at least 7 ADA holding no native
tokens.** Not 5, and not 6.

The builders set 5 ADA of that UTxO aside as script collateral, and the remainder has to come
back as its own output, which must itself clear the minimum for an output. A UTxO of exactly
5 ADA leaves nothing to return, and the build fails **reporting a shortfall on the change
output**, which reads as "you need more ADA" and is not what is wrong. Sending more ADA does not
fix it: the wallet in the report that produced this rule held 890 ADA.

If you hit it, the page offers a **Set up collateral** button that carves a clean 10 ADA box off
what you already hold (a payment from your wallet back to itself, so you keep every lovelace)
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

**Your four terms.** Under the prices are four boxes: **Spread (bps)**, **Book value cap (ADA)**,
**Daily loss limit (bps)** and **Reprice threshold (bps)**. They are the terms you sign in step 3,
and the keeper runs your book on exactly those numbers. Each starts at its default, and the page
refuses a value outside [the bounds on your terms](#the-bounds-on-your-terms). **Spread** starts
from your token's measured volatility, kept inside those bounds, and the page sizes your two
floors from it and the range you typed, so changing the spread changes the instance, the same way
changing a price does. What each term does is in
[What the statement says](#what-the-statement-says).

If you come back in the same browser you set it up from, the page restores the prices you chose
last time, because your address is built from them; today's market does not move them. From
another browser or device it starts from today's market, which builds a different instance;
use **Already have an instance?** to find the one you funded. If the market has since left that
range, the page names the side that would sit idle. Your inventory stays where it is and your
wallet can take it back at any time. That side simply will not trade until the market returns, or
until you move to a new instance with **Set up a new instance at today's market** (see
[If you are moving an existing book](#if-you-are-moving-an-existing-book)).

Press **Set up my instance**. The nine parameters are assembled and the address is derived on the
server, never in your browser, because the browser build of the Cardano library applies script
parameters differently and would produce a different address from the same inputs.

**Signs nothing. Costs nothing.** Afterwards you see your order address, with the line *"Nothing
rests there until you fund it."*

**Editing either price, or the spread, after this builds a different instance, at a new
address.** The page says so and names the address you are walking away from. The old one keeps
its own 2 ADA deposit and anything resting there; the new one costs another deposit. Put the
values back to reach the old instance again.

## 3 · Prove it's your wallet

One signature. No transaction, no fee, nothing submitted to the chain.

What you sign is a consent statement: fifteen lines of plain text in which you consent to
SaturnSwap making a market in your token, on terms the statement names. It ties your wallet, your
band, SaturnSwap's bot key and fee, your token and your four terms to this one ceremony, so nobody
can show you one ceremony and run another, or run yours on terms you did not sign. Press **Sign
with my wallet**, read the statement in your wallet's prompt, and sign. The signed
`possession-proof.json` then appears with a **Download** button.

### What the statement says

This is the statement at the default terms, with SaturnSwap's published values filled in. The
parts in angle brackets are yours:

```
SaturnSwap MMaaS consent, version 2
I consent to SaturnSwap making a market in my token with its bot key, on the terms below.
network: mainnet
your wallet: <your payout address>
our bot key: 1aba8f0a279e88d7aacd20a1f8e6d6ae4293a4f18fcb17cd43f20fd8
our fee: at most 20 bps (0.2%) of the ADA we pay out to your wallet, none when we close your order
your price limits: your book never buys above <bid ceiling> or sells below <ask floor> ADA per token
your token: <name> (policy <policy id>, asset name hex <asset name hex>)
token decimals: <decimals>
spread: 800 bps (8%) between your book's buying and selling prices
book value cap: 120 ADA (your ADA plus your tokens at our price); above it we return the book to your wallet
daily loss limit: 500 bps (5%) of your book's value at the start of each UTC day; past it we return the book to your wallet
reprice after our price moves: 150 bps (1.5%)
signed at: <the UTC time the page built the statement>
challenge: <64 hex characters>
```

Line by line:

- **network**, **your wallet**, **our bot key** and **our fee** are fixed by your ceremony. The
  bot key and the fee are two of
  [the five values SaturnSwap publishes](#the-five-values-saturnswap-publishes).
- **our fee** is the inventory validator's bound. The fee leg of a keeper transaction may take at
  most that share of the ADA the transaction pays out to your wallet, and nothing on a transaction
  that closes your order. The service fee drawn from your prepaid channel is a separate
  agreement; see [What it costs](README.md#what-it-costs).
- **your price limits** are your two floors from step 2. Your book never buys above the first
  price and never sells below the second.
- **your token** and **token decimals** name the one token your book quotes. The token line shows
  a readable name only when the asset name is 1 to 32 plain letters, digits, dots, hyphens
  or underscores; otherwise it gives the policy id and the asset name hex alone; for a token with
  an empty asset name it says `empty asset name`.
- **spread** is the full gap between your book's buying and selling prices. Each side sits half
  of it from our mid, and each is clamped into your price limits.
- **book value cap** counts both sides: your ADA plus your tokens at our price. Above it the keeper
  returns the book to your payout address. A rise in your token's price alone can take a book over
  it; see [The keeper can close your book](risks-and-exit.md#the-keeper-can-close-your-book).
- **daily loss limit** is a share of your book's value at the start of each UTC day, valued the
  same way as the cap. Past it the keeper returns the book to your payout address.
- **reprice after our price moves** is how far our feed mid has to move from the mid your resting
  quote was set at before the keeper pays to rebuild the quote. A change of terms never triggers a
  reprice by itself.
- **signed at** is stamped by the page's server clock when it builds the statement. It decides
  which of your statements governs; see [Which statement governs](#which-statement-governs).
- **challenge** is the hash of your nine parameters, so the statement is valid for this ceremony
  and no other.

The four terms must sit inside [the bounds on your terms](#the-bounds-on-your-terms), and the page
will not build a statement outside them. If a statement outside them ever governs a book, the
keeper does not quote it on nearer terms. It returns the book's order to your payout address once,
and a book with no order is simply not quoted.

**Your signature is published by you, in your own transaction.** It rides in the registration you
sign in the next step, as transaction metadata under label 8747. There is nothing to send us: no
Discord message, no email, no file transfer. The keeper reads it back off the chain and checks
three things. The signature must come from the payout address your ceremony baked in. The challenge
must be the one your nine parameters hash to. And the whole statement, rebuilt from your ceremony
and the values it names, must match what you signed byte for byte. A statement for anyone else's
ceremony, for an earlier version of yours, or spelled any other way, will not do.

**The earlier statement is not accepted.** Statements signed before version 2 were headed *proof
you hold the escape-hatch key*, never said "I consent" and named no terms. The keeper does not
quote a book whose only consent is one of those, and the page lists such an instance as *old
statement, not quotable*.

**Take the download anyway.** It is your own copy of what you agreed to, and it is the input the
independent verifier needs to judge your ceremony offline. The verifier prints the terms you
consented to.

**Sign this before you register.** The consent travels inside the registration transaction, which
is the most durable place for it, and the page puts this step ahead of registering for that
reason.

### What you can fund

Your terms bound the funding step. The page funds a book only when its value is at least
**50 ADA** and at most five sixths of your book value cap: **100 ADA** at the default cap of
120 ADA, and **50 ADA** at the lowest cap, 60 ADA. The value is the ADA in the order, its 2 ADA
min-UTxO included, plus your tokens at the page's two-feed mid, rounded up. If the page cannot read
a mid, it refuses to fund.

The cap is taken from the statement that governs your ceremony on chain, never from one you have
signed in this browser session and not yet published. The gap between the most you can fund and
your cap is headroom for a rise in your token's price, which raises your book's value with no
trade at all.

### Which statement governs

You can sign more than one statement for the same ceremony, to change your terms. Which one the
keeper runs is decided from the chain alone, and the page applies the same rules:

- Every statement for your ceremony counts, whether it rode in your registration or in a later
  transaction to your payout address. The keeper reads the whole history at your payout address,
  so a statement never ages out, however much that wallet is used.
- The statement with the newest **signed at** governs. Publishing an older statement again, by you
  or by anyone, changes nothing.
- A new statement takes effect when it is on chain, but no sooner than 24 hours after the previous
  one took effect. Until then the previous terms govern.
- A statement first put on chain more than 10 minutes before its own **signed at** time is refused
  for good. A statement the page builds and you publish straight away is never caught by this.
- Your token is pinned by the first statement that governs. A later statement naming another token
  never governs.
- Two different statements with the same **signed at** conflict. The keeper then returns your book
  to your payout address rather than choose between them.
- If the keeper cannot read your history in a round, it uses the terms it last verified or, having
  none, does not quote your book that round. It never falls back to an older statement.

### Changing your terms

Connect the same wallet and pick your instance under **Already have an instance?**. The panel
**Terms governing this instance** shows the terms in force. If you leave them as they are, no new
signature is needed.

To change them, edit the boxes and press **Review changed statement**, then **Sign the possession
proof**, then **Publish my consent**. Publishing builds one transaction: 2 ADA from your wallet
back to your own payout address, with the new signed statement as metadata under label 8747. The
2 ADA stays yours; you pay only the network fee. The page shows when the new terms take effect, and
funding stays closed until they govern. You cannot publish another change while one is waiting to
take effect.

Your price limits and your token do not change this way. They belong to the ceremony, and a new
band is a new instance; see [If you are moving an existing book](#if-you-are-moving-an-existing-book).

### If your credential is already registered without consent

A registration cannot be amended, so a credential registered without consent has no second
registration to carry it. When the registration panel finds your credential already registered,
and this tab has not sent a registration with your consent since the page was last loaded (a
reload forgets it), the page cannot tell whether the earlier registration carried one. Press
**Publish my consent**.

That builds the same one transaction as a change of terms: 2 ADA from your wallet back to your own
payout address, with your signed statement as metadata under label 8747. The page refuses to build
it unless the connected wallet holds the ceremony's escape-hatch key. Publishing is safe either
way. If your registration carried no statement, this one governs. If it carried one, the rules in
[Which statement governs](#which-statement-governs) decide, exactly as for a change of terms.

A statement published this way is exactly as valid as one carried in a registration. What makes it
valid is your signature, checked against your payout address and your nine parameters. The keeper
finds it by reading every transaction at your payout address, so it never ages out.

## 4 · Register it on chain

This is the transaction everything else waits on.

The panel first reads the chain to see whether this credential is already registered. If it
cannot read the chain it refuses rather than inviting you to pay a deposit blind, and re-checks
itself every 20 seconds, up to six times, before concluding this is an outage rather than indexer
lag and leaving the **Re-check** button to you.

You also choose, here and only here, what your own stake and governance weight do:

- **Stake pool**: **leave it blank.** Today a book stops being repriced at its first reward
  payout, so a delegated vault loses the service as soon as staking starts to pay; see
  [what changes the moment you delegate](risks-and-exit.md#what-changes-the-moment-you-delegate-to-a-pool).
  Blank is the default, and it means the vault's ADA stakes nowhere and earns nothing. SaturnSwap
  runs a stake pool and deliberately does not pre-fill it, because your stake is not ours to point.
- **Governance**: Abstain (default), No confidence, or a DRep you name. None of the three moves your
  ADA or lets anyone else spend it; a DRep votes, it never holds funds. Abstain is the default
  because it takes no governance position, not because your vault needs a DRep:
  [your vault's credential can withdraw with no DRep delegation at all](risks-and-exit.md#why-the-default-is-abstain-and-not-nothing),
  so none of the three changes what your book can do. If you name a DRep, the page
  checks that the ID is well formed and is a DRep ID rather than a committee key before your wallet
  opens. It does not check that the DRep is registered, so copy the ID from a directory.

Press **Register my credential (2 ADA deposit)** and sign in your wallet.

**You sign:** one Conway `RegisterAndDelegateCredential` certificate, witnessed by your
instance's own Plutus script. The ledger accepts it only with your signature.

**It costs:** a 2 ADA deposit, refundable in full when you retire, plus the network fee
(0.337761 ADA on the measured mainnet certificate).

**How to tell it worked:** the panel flips to *"Your credential is registered on chain"*, followed
by the certificate hash, a **View the certificate ↗** link and the line *"Funding can proceed
below."* If the indexers have not caught up you get the hash and the explorer link anyway, and
the panel keeps re-checking.

### If you are moving an existing book

When the page can tell you have replaced an earlier band, an extra step, **5 · Move your
existing order here**, appears between registration and funding, and funding renumbers to 6.
It moves your whole book, the token, the ADA and the three beacons, from the old instance to the
new one in a single transaction you sign. Nothing is burned or re-minted, and your inventory never
passes back through your wallet.

The move carries your current quote across; it does not reprice it. A leg changes only where the
new band forbids it: a sell price below the new ask floor is raised to that floor, and a buy price
above the new bid ceiling is lowered to that ceiling. The page names each leg it changed before you
sign. Once the move lands, the old instance is empty and the page offers **Reclaim my 2 ADA
deposit** for it, so moving costs roughly the network fees (see
[One band per instance](risks-and-exit.md#one-band-per-instance)).

## 5 · Put your inventory to work

The panel states the token you picked and how much of it you hold. Fill in:

- **Tokens to rest (ask side)**: in whole tokens; a **Use all** shortcut fills in your balance.
- **ADA to rest (bid side)**: optional. This is the ADA the keeper may buy with, inside your
  range. Leave it empty and your order can only sell until a sale gives it ADA to buy back with.

**How much to rest.** Between **50 ADA** of value and five sixths of your book value cap, counting
your token at today's price: at most 100 ADA at the default 120 ADA cap. The panel shows the floor
and the most you can fund under the statement that governs your ceremony, and refuses to build
outside them (see [What you can fund](#what-you-can-fund)). The keeper values your book again each
round it can price your token and returns it to your payout address above your cap, so a book
funded close to the most allowed can still be closed by an ordinary rise (see
[The keeper can close your book](risks-and-exit.md#the-keeper-can-close-your-book)). Start near the
floor, for that reason and the one in
[The order of operations that matters](#the-order-of-operations-that-matters).

**One order per book.** The keeper quotes one order at your order address. Any other order it
finds there is sent back to your payout address, up to two a day for each client, and any beyond
that rests unquoted. Fund once; to change the size, close your order and fund again. To make it
smaller, wait until 00:00 UTC before funding again; see
[Funding the same instance again the same UTC day](risks-and-exit.md#the-keeper-can-close-your-book).

Press **Build the funding transaction**. Nothing is signed yet. What you get back is:

- a plain-English verdict: one output to your derived order address resting *N* token units with
  *M* lovelace, exactly three beacons minted, the fee, and every other output returning to your
  own wallet;
- a **Download** button for `body.tx`, the exact unsigned transaction body;
- the command to check it yourself with `verify_create_body.py` (see
  [Checking it yourself](#checking-it-yourself)).

Build and sign are separate on purpose: you can run the source-available Python gate over the downloaded
body before you sign, so signing never rests on this page's word.

Then press **Sign and submit with my wallet**.

**You sign:** one cardano-swaps two-way create (your inventory into your order address, three
beacons minted, an inline datum carrying both your bid and your ask).

**It costs:** the network fee (0.293094 ADA on the measured mainnet create `dab1fd6a…`), plus the
2 ADA min-UTxO that rests inside the order with your inventory and comes back when you close it.

**How to tell it worked:** *"Submitted: `<hash>`. Your order goes live once this lands…"*, with a
**View the transaction ↗** link.

The registration check runs twice (once before anything is planned, and again immediately before
the signature), so a credential retired in between cannot let a funding through.

## The order of operations that matters

**Register before you fund.** Every owner action on a cardano-swaps order is authorised by a
withdrawal from your credential, and an unregistered credential cannot appear in a withdrawal. An
order funded at an unregistered credential can still be **filled by takers** while being
**repriced or closed by nobody, you included**, until somebody registers the credential. It is
stuck rather than lost: registration is permissionless, so anyone can register it for the 2 ADA
deposit. The page enforces the order anyway: funding refuses unless the chain positively says the
credential is registered, and it refuses when the chain cannot be read.

{% hint style="danger" %}
**Never send funds to your order address with an ordinary wallet transfer.** It is a script
address. A plain send arrives with no datum, and the validator decodes the datum before it
reaches any branch, so the transfer is permanently unspendable, by SaturnSwap, by you, and by
the wallet that owns it. The only safe funding is the transaction step 5 builds, which attaches
the beacons and the price datum. A book holds one order; to change its size, close the order and
run step 5 again. To make it smaller, wait until 00:00 UTC before funding again; see
[Funding the same instance again the same UTC day](risks-and-exit.md#the-keeper-can-close-your-book).
{% endhint %}

**Close your orders before you retire the credential.** See
[Retire the credential](risks-and-exit.md#id-2.-retire-the-credential-and-take-the-2-ada-back).

**A freshly funded order rests at the edge of your range** until the keeper's first reprice moves
it. That is the outermost price you authorised, and it is a gift to whoever trades against it.
It cost SaturnSwap 14.6 ADA on a live mainnet order, found deliberately on its own money. Fund
near the 50 ADA floor first and let it be worked before you commit size.

## After funding

Once the create lands, your order is a live two-way order on SaturnSwap's book. Anyone can fill
it at your posted prices, whether or not the keeper is working it yet.

What the keeper does once your book is enrolled: each round it reads a mid from the two feeds and
quotes your book at the spread you signed, each side half the spread from the mid and clamped into
your band. At the default 8% spread, the ask sits 4% above the mid and the bid 4% below it. It
rebuilds your order only when the mid has moved at least your reprice threshold (1.5% by default)
from the mid your resting quote was set at, so smaller moves leave your quote where it is. Whatever
the threshold, a resting ask at or below the mid, or a resting bid at or above it, is repriced.
SaturnSwap signs and pays the network fee for every one of those reprices.

The keeper also returns your book to your payout address above your book value cap and past your
daily loss limit. See
[The keeper can close your book](risks-and-exit.md#the-keeper-can-close-your-book).

### How to check your book is live

Come back to [saturnswap.io/v3/mmaas](https://saturnswap.io/v3/mmaas) and connect the same
wallet. Under **Already have an instance?** the page finds your instances from the chain (the
ceremony's own validator carries the numbers it was built with, so there is nothing to look up or
type) and lists each one roughly like this:

```
44.00 ADA · 0.0891 - 0.1094 ADA · last worked 3h ago · 3c265036cwm9…
```

An instance whose only consent is a statement from before version 2 carries the label *old
statement, not quotable*.

**"last worked"** is the answer you want. It is the age of the resting UTxO, read from chain
alone, and it asks the keeper nothing: a refused ceremony, a missing config entry and a stopped
process are indistinguishable from outside, which is exactly why the check does not try to tell
them apart. A number that keeps climbing means nobody is repricing you.

You can compute the same number yourself from any indexer, or just watch your order address on a
block explorer. Your settled volume, fills and fees are also on your own dashboard at
[saturnswap.io/v3/mmaas/book](https://saturnswap.io/v3/mmaas/book): sign in with the wallet your
book was set up with. It shows figures once SaturnSwap has linked that wallet to a billing grant;
until then it says no market was found for the wallet.

If this wallet has no funded instance, the page lists any instance you registered but never funded
as set up and waiting for inventory. It still holds its 2 ADA deposit. To use it, open the page in
the browser you set it up from and pick its token: the page restores the range and spread you last
used for that token there. Elsewhere, entering the same range can build a different instance,
because the floors are sized from the range and the spread, and the spread box starts from the
token's measured volatility on the day. That instance charges another 2 ADA deposit. The page does not yet offer a retire step for an
instance that was never funded, so that deposit stays with the instance until it does.

### Finding an instance by its two floors

The list is built from your registration transactions. If one cannot be read, **Enter the floors
by hand instead** takes the two floors as fractions, `min_asset2_price` and `min_asset1_price`,
exactly as they appear in your params file, plus your token's decimals, and derives the instance
they describe.

- **If nothing rests there,** the page says so. Those floors describe a different instance from
  the one you funded. Check them against your params file before you sign anything.
- **Decimals do not change which instance you land on;** the two fractions decide that. Decimals
  appear in the message you sign, so a wrong value produces a proof the verifier refuses, not one
  that points somewhere else.
- **The page offers the possession signature only after the chain confirms a book rests there.** A
  signature for an instance that holds nothing looks exactly like a signature for one that does.

## Checking it yourself

Two source-available Python tools ship in the public
[saturnswap-maker-verify](https://github.com/Flux-Point-Studios/saturnswap-maker-verify) repo,
and both work from a fresh clone with nothing from us:

- **`verify_create_body.py`** judges the funding transaction body before you sign it. The
  funding panel prints the exact command and offers `body.tx`.
- **`verify_ceremony.py`** rebuilds the validator from source, re-applies your nine parameters,
  derives the script hash and both addresses, and checks your possession proof against them. For
  a version 2 statement it prints the terms you consented to.

`verify_ceremony.py` will not judge a ceremony until someone has proved they can sign for the
escape-hatch key; without that proof it would only confirm that SaturnSwap's arithmetic agrees with
itself. If anything SaturnSwap told you disagrees with what the source derives, it exits loudly,
names the difference, and prints *"DO NOT FUND THIS ADDRESS"*. So do not fund an order address on
the page's word: the page derives it from your parameters, and the verifier re-derives it from the
published source. Clone the verifier yourself rather than taking a copy from anyone (see
[Check an existing address](validator-generations.md#check-an-existing-address)).

Both take `--params my-ceremony.params.json`. **The guided flow does not hand you that file**,
only the possession proof. Two ways to get it:

- Open **Set this up by hand, from a cold key** on the same page and enter the two floors the
  guided flow showed you (*"Never buy above X, never sell below Y"*) plus your token's decimals.
  The expert form converts them with the same functions the guided flow does, so it reproduces the
  identical nine parameters and prints the file. The guided flow fills in seven of the nine for you
  (five published by SaturnSwap, two read from your wallet); the expert form lets you set your four
  values yourself (escape-hatch key, payout address and both floors) and shows the five published
  ones for you to check, which is what you want if your escape-hatch key lives somewhere this
  browser will never see. The address is still derived on SaturnSwap's server from the values you enter, for the reason
  in step 2.
- Recover them from the chain: the applied validator carries all nine and rides in the witness
  set of your own registration transaction.

Keep that file. It is nine public values, and it is the simplest input to the recovery tool
described in [Leaving if SaturnSwap is gone](risks-and-exit.md#leaving-if-saturnswap-is-gone).

### Checking a params file you already have

**Already set up? Check what your vault bound** on the page takes a params file or ceremony receipt
and lists what it claims: the bound script hash, your floors, your payout address, your
escape-hatch key and the five published values. It then gives you the verifier command. The list is
only an echo of the file. The check is the command, run on your own machine against the public
source; the page plays no part in it.

### The five values SaturnSwap publishes

Five of the nine parameters behind your vault are SaturnSwap's to publish and yours to check. The
page fills them in so you cannot mistype one, and a mistake would be permanent: each value is an
input to your address. This table is a second place to check them, outside the page. Compare the
values in your params file against it, and confirm each one on chain the way the last column
says. The verifier takes `dapp_hash` and `beacon_id` as given, so this check is yours to do.

| Parameter | Published value | What it is | How to check it |
|---|---|---|---|
| `fee_address` | `addr1v9wr69p2tx8dx2lat8rzznahxh4xhfl075yzm8uxmth4tvcf3lx47` | The only address the inventory validator's fee leg may pay. | An enterprise mainnet address (header `0x61`) with payment key hash `5c3d142a598ed32bfd59c6214fb735ea6ba7eff5082d9f86daef55b3`. It receives this fee and nothing else: no change, no payouts, no treasury. |
| `fee_bps` | `20` | The most the inventory validator's fee leg may take, 0.20% of what the transaction pays out to you, and nothing on a transaction that closes an order. This is not the service fee drawn from your prepaid channel; see [What it costs](README.md#what-it-costs). | The validator declares its own ceiling, `const max_fee_bps = 500`. An instance built with a higher rate rejects every bot action. |
| `adam_bot_pkh` | `1aba8f0a279e88d7aacd20a1f8e6d6ae4293a4f18fcb17cd43f20fd8` | The one key SaturnSwap holds against your instance. It can reprice and cancel. Your value can only land at your order address, your payout address or the bounded ADA fee leg. It is also the `our bot key` line of the statement you sign. | The key funds its own enterprise address, `addr1vydt4rc2y70g34a2e5s2r78x66hy9yay7x8uk97dg0eqlkqefwst2` (header `0x61`), whose payment credential is this hash. The keeper pays for every reprice from that address, so the transactions spending from it are this key's signatures, and they grow every day the keeper runs. Check the credential. |
| `dapp_hash` | `11928a3ac3b65edbf103ea6bb3362e39b879a36f02897df31c40917b` | The two-way cardano-swaps validator your orders rest against. Only the two-way order datum carries both a bid ceiling and an ask floor. | The beacon policy below commits to it: fetch that policy's script from any mainnet indexer and this hash appears inside it as an applied parameter. You can also rebuild it from source; see [What it actually does](risks-and-exit.md#what-it-actually-does). |
| `beacon_id` | `8a199a17ef4517215945aaf3c8c5204c60fd94d34c46d341e99c8fcf` | The policy that marks your orders on the book. | Fetch its script from any mainnet indexer and read its error strings: *"Two-way swaps must have exactly three kinds of beacons"*, *"Wrong asset1_beacon"* and *"Wrong asset2_beacon"*. A one-way policy says *"One-way"* and *"Wrong offer_beacon"* instead. That is the only way to tell the two deployments apart. |

### The bounds on your terms

The four terms in your statement are yours to choose, inside these bounds. The page refuses a value
outside them, and the keeper never quotes one: it returns the book instead (see
[What the statement says](#what-the-statement-says)). The bounds may be widened later, which
leaves every signed statement valid. They are not narrowed once a client has signed, because a
narrower bound would return every book signed outside it.

| Term | Default | Lowest | Highest | Unit |
|---|---|---|---|---|
| Spread | 800 (8%) | 400 (4%) | 6000 (60%) | basis points, the full gap between the buying and selling prices |
| Book value cap | 120 | 60 | 120 | whole ADA, your ADA plus your tokens at our price |
| Daily loss limit | 500 (5%) | 100 (1%) | 5000 (50%) | basis points of your book's value at the start of each UTC day |
| Reprice threshold | 150 (1.5%) | 150 (1.5%) | 1000 (10%) | basis points of our feed mid |

Two more rules:

- **The spread must be more than twice the reprice threshold.** Each side rests half the spread
  from the mid it was set at, and the keeper leaves it there until our mid has moved by the
  threshold. The rule keeps your bid below the market and your ask above it for every move the
  keeper does not follow. The defaults pass: 2 x 150 = 300, under 800.
- **Token decimals are 0 to 18.**

Funding has its own bounds, set from the terms that govern your ceremony on chain:

| Funding bound | Value |
|---|---|
| Floor | **50 ADA** of book value |
| Most you can fund | five sixths of your book value cap, rounded down to the lovelace: **100 ADA** at a 120 ADA cap, **50 ADA** at a 60 ADA cap |

## What is not automatic yet

Stated plainly, because acting on a stale claim here costs real money:

- **Enrolment is automatic, up to five books.** Every version 2 statement names its token, the
  expert form's included, and the keeper finds a book with a governing statement on the chain and
  starts quoting it without anyone at SaturnSwap adding it. One keeper quotes at most five books,
  and SaturnSwap's own NIGHT book holds one of the five.
  A book that arrives while five are enrolled is not quoted: its order is returned to your payout
  address, and it can enrol again once a seat is free. Until the keeper's first reprice, your order
  rests and is fillable by takers at whatever price it last carried.
- **The guided flow does not give you your parameters file.** See above for the two ways to
  obtain it.
- **The client dashboard shows figures only after SaturnSwap links your wallet to a billing
  grant.** Until then it says no market was found for your wallet, and "last worked" on the
  re-entry panel and a block explorer are the honest checks.

None of these can take your funds, and none of them can stop you leaving: every exit is
authorised by your signature alone.

Next: [Risks, limits, and how to leave](risks-and-exit.md)
