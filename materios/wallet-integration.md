# Wallet & SDK Integration

Materios is a Substrate chain, but it is **not** a stock Substrate chain. Two deviations
will break an integration that assumes the defaults, and both fail in ways that are easy
to misdiagnose. Read this page before writing signing code.

Everything below was verified against live preprod on 2026-08-25.

## Connection facts

| Field | Value |
|---|---|
| RPC (WebSocket) | `wss://materios.fluxpointstudios.com/preprod-rpc` |
| Chain | `Materios Preprod v6` |
| Genesis hash | `0x0e46e33f639a56cc8780fd871d9a15e16d99af248526f907cb560cb40849f7bf` |
| Spec version | 235 |
| **Transaction version** | **4** |
| Block time | 6 seconds |
| Finality | GRANDPA |
| `system_properties.ss58Format` | 42 |
| `tokenSymbol` / `tokenDecimals` | `MATRA` / 6 |

> **Preprod is a public testnet.** The native token here is tMATRA and has no economic
> value. Mainnet parameters will differ; do not hardcode preprod values into a build you
> intend to point at mainnet.

## Deviation 1 — the fee extension is `ChargeMotra`, not `ChargeTransactionPayment`

This is the one that breaks most clients.

Materios pays transaction fees in **MOTRA**, not MATRA, through a custom pallet. The
runtime's signed extensions are:

```
CheckNonZeroSender, CheckSpecVersion, CheckTxVersion, CheckGenesis,
CheckMortality, CheckNonce, CheckWeight, ChargeMotra
```

A stock Substrate chain ends that list with `ChargeTransactionPayment`, which carries a
`Compact<Balance>` tip. **`ChargeMotra` carries no tip field.** There is nothing to encode
in that position.

### What breaks

Any client that hardcodes the default extension set — rather than reading the extension
list out of chain metadata — will append a `Compact<tip>` that the runtime does not
expect. The extra bytes shift the payload, the signature fails to verify, and **every
transaction is rejected**, not just tipped ones.

| Client | Behaviour |
|---|---|
| **polkadot-js** | Works. It builds the extension list from metadata, so it picks up `ChargeMotra` automatically. |
| **subxt** | Breaks unless you define a custom `SignedExtra`. Its default assumes `ChargeTransactionPayment`. |
| **txwrapper-core** | Breaks. Its transaction construction assumes the default set. |
| **py-substrate-interface** | Breaks on the tip path. Do not pass a `tip` argument. |

There is also **no `TransactionPaymentApi`** on this runtime. `payment_queryInfo` returns
`Method not found`, so **`tx.paymentInfo()` throws** — do not call it to estimate fees, and
do not treat the throw as a connectivity failure.

### What to do

1. **Build the signed extension set from chain metadata.** Never hardcode it. This is the
   single change that makes most clients work.
2. **Never set a tip.** There is no field for one. Omit the parameter entirely rather than
   passing zero.
3. **Do not call `paymentInfo()`.** Fee estimation goes through MOTRA; see below.

## Deviation 2 — the SS58 prefix is reported two different ways

The chain reports two different SS58 prefixes depending on where you look:

- `system_properties.ss58Format` → **42** (the generic Substrate prefix)
- the runtime constant `System::SS58Prefix` → **0**

Both refer to the same 32-byte public key. They produce **different address strings** for
the same account, so a tool reading one source and a tool reading the other will disagree
textually while being in perfect agreement about the underlying key.

This bites hardest in comparisons: an address that looks completely different from the one
in your database may be the same account. A string equality check across two tools that
chose different prefixes silently returns "no match" forever.

### What to do

- **Treat the 32-byte public key as the account identity**, not the encoded string.
- **Normalise before comparing.** Decode to the public key, then re-encode to one prefix.
- **Display with prefix 42.** That matches `system_properties`, the explorer, and the
  gateway, so it is what users will see elsewhere.

```javascript
import { decodeAddress, encodeAddress } from '@polkadot/util-crypto';

// Same key, either prefix in -> one canonical form out.
const canonical = (addr) => encodeAddress(decodeAddress(addr), 42);
```

The prefix split is a known inconsistency and is queued to be reconciled in a future
runtime upgrade. Until then, normalise.

## The two-token model

| Token | Decimals | Transferable | Role |
|---|---|---|---|
| **MATRA** | 6 | Yes | Staking, governance, transfers, balances |
| **MOTRA** | 15 | **No** | Transaction fees only |

MOTRA is a capacity token, not a currency. It is **generated passively from the MATRA an
account holds**, decays if unused, and is burned when spent on fees. It cannot be
transferred between accounts. The pattern mirrors NIGHT generating DUST on Midnight.

Consequences for a wallet:

- **`tokenDecimals` in `system_properties` is 6 — that is MATRA only.** MOTRA's 15 decimals
  are not advertised there. Formatting a MOTRA balance with 6 decimals overstates it by a
  factor of 10⁹.
- **An account with MATRA but no MOTRA yet cannot transact.** MOTRA accrues over blocks. A
  freshly funded account needs to wait before its first transaction, which looks like a
  bug if you are not expecting it.
- **A zero MATRA balance means MOTRA generation has stopped.** Fee capacity is downstream
  of the MATRA holding, so a fully swept account cannot pay to move anything.
- **Fees never touch the MATRA supply.** MOTRA is consumed; no MATRA is burned or minted by
  fee flow.

## Verification checklist

Run these against a real connection before declaring an integration working. Each one
targets a specific way this chain differs from the default.

```javascript
import { ApiPromise, WsProvider } from '@polkadot/api';

const api = await ApiPromise.create({
  provider: new WsProvider('wss://materios.fluxpointstudios.com/preprod-rpc'),
});

// 1. You are on the right chain. Compare against the genesis hash above.
console.assert(
  api.genesisHash.toHex() ===
    '0x0e46e33f639a56cc8780fd871d9a15e16d99af248526f907cb560cb40849f7bf',
  'wrong chain',
);

// 2. The runtime is what you built against.
const v = api.runtimeVersion;
console.log('spec', v.specVersion.toNumber(), 'tx', v.transactionVersion.toNumber());

// 3. THE IMPORTANT ONE: your client resolved ChargeMotra, not ChargeTransactionPayment.
//    If this prints ChargeTransactionPayment, your signing WILL fail — the client is
//    using the stock extension set instead of reading metadata.
console.log('signed extensions:', api.registry.signedExtensions);

// 4. Sign and submit one real extrinsic. A dry-run does not exercise the extension
//    encoding, so it will not catch the tip-byte failure. Only a submitted, included
//    transaction proves the signing path.
```

Step 4 is not optional. The `ChargeMotra` failure is an encoding mismatch that only
surfaces when the runtime verifies a real signature — every offline check passes right up
until submission.

## Getting test funds

The gateway exposes a faucet for preprod:

```bash
curl -X POST https://materios.fluxpointstudios.com/preprod-blobs/faucet/drip \
  -H 'content-type: application/json' \
  -d '{"address":"<your SS58 address>"}'
```

One drip per address. A second call returns `Address already received a drip` rather than
topping up. After the drip, allow a few blocks for MOTRA to accrue before your first
transaction.

## Where to go next

- [Materios overview](README.md) — architecture, pallets, current network parameters
- [Node requirements](node-requirements.md) — running your own node or RPC endpoint
- [Cardano L1 anchoring](cardano-anchoring.md) — how Materios state settles to Cardano

If you hit something this page does not cover, open an issue on
[Flux-Point-Studios/docs](https://github.com/Flux-Point-Studios/docs) — integration
friction is worth documenting for the next team.
