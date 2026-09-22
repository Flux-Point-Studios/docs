# Validator generations

An order address commits to the **applied** validator: source plus the client's nine
ceremony parameters. A source generation hash identifies the **unapplied** source;
it is not a client's stake credential. Publishing a new generation cannot upgrade an
existing address or move its funds.

## The list lives in the verifier, not here

**[Validator generations — saturnswap-maker-verify/GENERATIONS.md](https://github.com/Flux-Point-Studios/saturnswap-maker-verify/blob/main/GENERATIONS.md)**

That file is the only place the published generations are enumerated, and this page
deliberately does not repeat them. A copy of that list on a docs site is a copy that
goes stale the moment a generation is cut — and a stale copy is worse than none,
because a client whose book runs the newest generation reads an older list, does not
find their hash, and concludes their source was never published. That page sits beside
the source it describes and beside the tests that check it lists exactly what the
repository ships, so it cannot say something the repository cannot do.

Which generation applies to YOUR address is decided by your credential, not by which
is newest. The verifier finds it for you.

## Check an existing address

Start with the ceremony JSON and order address you actually funded. Use the command in
[the public verifier README](https://github.com/Flux-Point-Studios/saturnswap-maker-verify#readme),
including your expected order address, independent price-band check, and wallet
possession proof. For a ceremony on any generation other than the current one, add the
selector named in GENERATIONS.md:

```sh
--project ./generations/<the source generation in that table>
```

Run the **root** `verify_ceremony.py` with that option. It rebuilds the selected source,
compares it with its committed blueprint, applies the parameters, and compares the
result with your expected address. A historical directory contains source artifacts,
not a second copy of the CLI. Do not change an expected address to make a mismatch pass.
Do not substitute another generation's possession proof: its challenge also changes.
`--derive-only` proves neither wallet possession nor live chain state.
Seven-parameter and other unlisted generations need their own matching source release;
the nine-parameter packages do not verify them.

## What the distinction means

Generations differ in what the bot is allowed to do with a client's order, and
GENERATIONS.md states the differences per generation, in the table. Broadly: the
earliest published generation conserves assets during a bot transaction but does not
bind a continuation's declared trading pair to the spent one, nor exclude withdrawn
staking rewards from the fee basis; later ones close those, and the current one also
requires the bot's continuation to carry the ceremony's own beacons and to declare no
expiration.

These are generation-specific guarantees, not claims of a complete audit. The band is
an immutable bid ceiling and ask floor, not an oracle-relative price guarantee. Public
taker fills run the DEX's trading rules and are distinct from bot owner actions.

Moving an old book to the current generation requires a new ceremony and address, and
client-authorized recovery and funding transactions. The client-signature branch
remains available on every generation; changing the verifier does not migrate a book.
The included CLI escape helper uses a zero withdrawal and refuses a nonzero or
unreadable reward balance by default. Verification support is not a claim that this
helper can recover every delegated account without additional client-signed steps.
