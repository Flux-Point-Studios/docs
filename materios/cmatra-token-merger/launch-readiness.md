---
description: >-
  Launch date, audit posture, rate-table reference, and wallet compatibility
  for the cMATRA Token Merger.
---

# Launch Readiness

***

## Launch Date

**May 28, 2026.** The redemption window opens on this date.

* The redemption window is **6 months** long. It closes on **November 28, 2026**. See [README → Window mechanics](README.md#window-mechanics) for the full window rules and [FAQ Q35](faq.md#35-what-happens-after-six-months) for what happens at expiry.
* The live redemption portal is at [fluxpointstudios.com/matra-merger](https://fluxpointstudios.com/matra-merger).
* The launch is shipping under the **pre-audit posture** disclosed below — the third-party security audit is still in progress. Review [Audit Status](#audit-status) and the [risk disclosures](legal-and-disclaimers.md#risk-disclosure) before redeeming.

***

## Audit Status

**Security audit in progress. Pre-audit launch posture.**

* The Materios chain runtime, the merger contracts, and the cMATRA mint/burn scripts are under active third-party security review.
* The full audit report will be linked here once published.
* Pre-audit safeguards in effect during the redemption window:
  * Multisig-controlled administrator key with explicit pause and throttle authority (see [Legal § Force Majeure](legal-and-disclaimers.md#force-majeure-and-emergency-pause)).
  * Public testnet running the same code path that handles the merger on mainnet.
* Findings that materially affect users will be announced via [Discord](https://discord.gg/MfYUMnfrJM) and this page.
* Material status updates during incidents are posted in [Discord](https://discord.gg/MfYUMnfrJM). Support is best-effort during the redemption window; no service-level agreement is offered. Critical incidents (chain halt, contract exploit) are triaged immediately.
* Do **not** post seed phrases, raw private keys, or full wallet contents in any support channel. Materios staff will never ask for them.

***

## Rate Table

The reference rates are published on the live portal: [fluxpointstudios.com/matra-merger](https://fluxpointstudios.com/matra-merger).

The rate table is locked for the full 6-month redemption window once published, except under the governance-approved correction clause in [README → Window mechanics](README.md#window-mechanics). Source code and historical rate-table revisions are public at [github.com/Flux-Point-Studios/matra-token-merger](https://github.com/Flux-Point-Studios/matra-token-merger).

***

## Window Length

* **6 months** from the launch date.
* Public surrenders are permanently disabled at the on-chain validator level once the deadline passes. Any unreleased cMATRA returns to the project treasury under [README → Deadline and After-Window Handling](README.md#deadline-and-after-window-handling).
* No extension or exceptional late-surrender mechanism is planned. Holders are encouraged to redeem well before the deadline.

***

## Where cMATRA Lands

* **Cardano-native at launch.** cMATRA is minted on Cardano under the merger policy; you receive it in the same Cardano wallet that signed the surrender transaction.
* **Materios bridge is future work.** The bridge that moves cMATRA into MATRA on the Materios partner chain is not yet operational. See [FAQ Q37](faq.md#37-do-i-need-to-bridge-to-materios-immediately-after-redeeming) for the intended onboarding flow (Cardano-first, address association, bridge or lock into Materios later as the flow matures).

***

## Wallet Compatibility

The redemption frontend uses **CIP-30** for wallet connection. The following wallets are supported at launch:

| Wallet                | Support type   |
| --------------------- | -------------- |
| Eternl                | CIP-30         |
| Lace                  | CIP-30         |
| Nami                  | CIP-30         |
| Flint                 | CIP-30         |
| Begin                 | CIP-30         |
| Vespr                 | CIP-30         |
| Typhon                | CIP-30         |
| NuFi                  | CIP-30         |
| Yoroi                 | CIP-30         |
| GeroWallet            | CIP-30         |
| Ledger (hardware)     | via supporting CIP-30 wallet (e.g. Eternl, Lace) |
| Trezor (hardware)     | via supporting CIP-30 wallet (e.g. Eternl, Lace) |

* Hardware wallets are connected through a CIP-30 software wallet that supports the device.
* Other CIP-30-compliant wallets that are not on this list may still work at the protocol level, but Materios only QAs the redemption frontend against the wallets above.
* If you are unsure whether your wallet supports CIP-30, check the wallet's own documentation before the launch date.

***

## Pre-Launch Checklist for Holders

Before the redemption window opens:

1. **Self-custody.** Move eligible legacy assets (AGENT, SHARDS, eligible NFTs) into a CIP-30 wallet you control. Exchange and custodial balances are not directly redeemable; see [FAQ Q27](faq.md#27-what-if-my-assets-are-on-a-centralized-exchange-or-in-someone-elses-custody).
2. **Unfarm.** Remove AGENT/SHARDS from LP positions, lending protocols, and other script-controlled positions. LP tokens do not redeem — the underlying does. See [FAQ Q24](faq.md#24-what-if-my-agent-or-shards-are-in-an-lp-farm-on-minswap-wingriders-or-another-defi-platform).
3. **De-list NFTs.** Withdraw eligible NFTs from marketplaces and scripts. The eligible NFT itself (the user token for CIP-68) must be in your wallet at the time you surrender it. See [FAQ Q32](faq.md#32-what-if-my-nft-is-listed-on-a-marketplace-or-held-in-a-script).
4. **Read the legal page.** [Legal & Disclaimers](legal-and-disclaimers.md) — especially the **Jurisdiction Status** section.
5. **Read the FAQ.** [cMATRA FAQ](faq.md) covers the questions most holders ask before redeeming.

***

**Version:** 1.0 | **Launch:** May 28, 2026 | **Window closes:** November 28, 2026\
**Owner:** Flux Point Studios.
