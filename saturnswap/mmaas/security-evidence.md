# Security evidence and its scope

We run an internal red team against MMaaS every week using Claude, Codex and Kimi models.
The work attacks the protocol, investigates findings and checks for regressions. We choose
ongoing adversarial review as the protocol changes. **There is no third-party audit.**
This process is internal evidence, not external certification or a guarantee that no
vulnerabilities remain.

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

The tested current generation's **unapplied** validator hash is
`18d2246d8b552b9e462ec93dece5716a7154314680b3f326a854789d`.
Applying the nine ceremony parameters derives each book's different credential and addresses.
Match those against your funded address using the public verifier; matching a source revision
alone is insufficient.

Existing books using the older unapplied hash
`adc2a7f19bf63b378c06c7d941bba6b7f6312cb8cce5b153f356efe4`
retain that generation's code. The newer pair-preservation and staking-reward fee protections
do not appear in an old book merely because source or documentation changed. See
[Validator generations](validator-generations.md).

We make no Z3 or other formal-proof claim for the active validator without a reproducible
artifact naming the exact validator hash, assumptions, properties proved and verifier command.
Unit tests, generated property cases and mutation checks are valuable evidence with their own
scope; they are not exhaustive formal proofs.
