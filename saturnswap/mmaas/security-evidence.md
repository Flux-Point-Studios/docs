# Security evidence and its scope

Point-in-time audits are not enough for code that changes every week, so MMaaS is attacked
continuously and every result is dated. An AI red team attacks the validator and the keeper. What it
finds gets fixed and re-attacked before it ships. A full combined run (validator, keeper, backend and
page) is repeated before public sign-up opens, then every week.

- **Weekly record.** The validator and the keeper are attacked every week, and every run is published
  with its date, including the runs that find nothing. The latest is [Red team 2026-10-19](https://github.com/Flux-Point-Studios/docs/issues/50).
- **Combined runs.** The most recent combined run to reach a verdict, on September 10, 2026, returned
  BLOCK: findings to fix before launch. Funding new books through the page stays closed until a
  combined run against the release that will take funds returns GREEN.
- **Validator tests.** Unit and property tests are recorded against a named source revision (below).

None of this is a guarantee that no vulnerabilities remain.

## Recorded validator tests

The September 9, 2026 internal run at contract source revision
`44714f46a6d4c1f1296f48c98d55bcc2c7090df1` passed **122 unit tests and five property
tests with 1,000 generated cases each**. The property additions did not change the
production validator or compiled blueprint.

| Property | What it checks |
|---|---|
| Conservation | Generated inventory values cannot fund an unauthorised skim. |
| Pair preservation | Splitting continuations cannot substitute a different input asset pair. |
| Rewards and fees | Withdrawn rewards remain conserved and are excluded from the inventory fee basis. |
| Client exit | The client-authorised path still works with invalid bot price floors. |
| Fill/reprice sequence | Modeled fill continuations and reprices preserve the tested client exit properties. |

Removing the pair-preservation check or reward-fee exclusion caused the corresponding tests
to fail in mutation runs. These results show that those tests detect those removed checks.
They do not establish coverage of every possible defect.

**The fill-sequence properties model fills; they do not execute the separate DEX spend
validator.** The generated inputs cover the defined model, not every Cardano transaction
or a complete interaction between both validators.

The record above is from the internal contract repository. It is an internal test report,
not a publicly reproducible formal-proof artifact. The
[public verifier](https://github.com/Flux-Point-Studios/saturnswap-maker-verify)
provides the validator source and ceremony/address verification path.

## Bind evidence to the code your book uses

Evidence binds to a **generation**, and a book keeps the generation its credential hashes.
Which one that is, and which unapplied hash identifies it, is stated in
[GENERATIONS.md](https://github.com/Flux-Point-Studios/saturnswap-maker-verify/blob/main/GENERATIONS.md)
beside the source itself, and not here, where a copy would go stale the moment a generation is
cut and would then deny a real book's existence.

Applying the nine ceremony parameters derives each book's different credential and addresses.
Match those against your funded address using the public verifier; matching a source revision
alone is insufficient.

A protection added in a later generation does not appear in an older book merely because
source or documentation changed. See [Validator generations](validator-generations.md).

We make no Z3 or other formal-proof claim for the active validator without a reproducible
artifact naming the exact validator hash, assumptions, properties proved and verifier command.
Unit tests, generated property cases and mutation checks are valuable evidence with their own
scope; they are not exhaustive formal proofs.
