---
description: >-
  Live launch-status dashboard for the cMATRA Token Merger. Audit posture,
  support channels, rate-table reference, wallet compatibility, and the
  current launch date.
---

# Launch Readiness

This page is the single-pane status board for the cMATRA Token Merger. Anything that needs a human update before launch is marked **TBD** so it is grep-able from CI.

<!-- LAUNCH OWNER: keep this page authoritative. The README and FAQ link
     here for the live launch date, audit status, and support channel.
     If those move, update this page first, then push the link forward. -->

***

## Launch Date

**TBD — to be announced.**

* The redemption window is **6 months** long, measured from the launch date published here. See [README → Window mechanics](README.md#window-mechanics) for the full window rules and [FAQ Q35](faq.md#35-what-happens-after-six-months) for what happens at expiry.
* The launch date is published here only after:
  * Legacy reward materialization is reconciled (see [FAQ Q21](faq.md#21-why-are-final-fungible-rates-not-frozen-today)).
  * Final Team treasury waivers are published.
  * The final fixed rate table is signed off.
  * The security audit is complete or the pre-audit launch posture is explicitly disclosed (see [Audit Status](#audit-status) below).
  * The legal sign-off on [Legal & Disclaimers](legal-and-disclaimers.md) is complete (jurisdiction, KYC threshold, prohibited-jurisdiction list published).

Until **all** of the above land, the launch date stays **TBD** on this page.

***

## Audit Status

**Security audit in progress. Pre-audit launch posture.**

* The Materios chain runtime, the merger contracts, and the cMATRA mint/burn scripts are under active third-party security review.
* The full audit report will be linked here when it is published: **TBD — audit report URL**.
* In the interim, Materios is operating a pre-audit launch posture:
  * Multisig-controlled administrator key with explicit pause/throttle authority (see [Legal § Force Majeure](legal-and-disclaimers.md#force-majeure-and-emergency-pause)).
  * Internal peer review across the FPS engineering team and external advisors before any live runtime upgrade or deployment.
  * Public testnet (`materios_preprod_v6`) running the same code path that will be deployed for the merger.
* Findings that materially affect users will be disclosed in the audit report and announced through the FPS Discord and this page.

***

## Support Channel

**Discord: TBD — [link to FPS Discord with #cmatra-merger-support channel].**

* For urgent merger-specific issues (failed redemption, missing balance, transaction stuck on Cardano), open a ticket in the FPS Discord under **#cmatra-merger-support** once the channel is live.
* No service-level agreement is offered. The support team is best-effort during the redemption window. Critical operational incidents (chain halt, contract exploit) are triaged immediately under the standard incident-response process.
* Material status updates during incidents will be posted on the FPS Discord status channel and (where appropriate) on this page.
* Do **not** post seed phrases, raw private keys, or full wallet contents in any support channel. Materios staff will never ask for them.

***

## Rate Table Reference

* The canonical fixed rate table is committed in the `matra-token-merger` repository at:

  ```
  audit_pack/2026-04-19/rate_table_cmatra.json
  ```

* This file is the byte-exact source the on-chain mint reads from. The display rates rendered in [README → What you get](README.md#what-you-get-reference-rates--april-19-2026) and [FAQ → 10a](faq.md#10a-what-changed-vs-v33) are derived from this file with rounding for human readability.
* All seven asset buckets sum to exactly **722,500,000 cMATRA** in base units (verified at publish time in `audit_pack/2026-04-19/`).
* The rate table is locked for the full 6-month redemption window once published, except under the documented governance-approved correction clause in [README → Window mechanics](README.md#window-mechanics).

***

## Window Length

* **6 months** from the launch date.
* The `ProcessSurrender` spending path on the Cardano-side merger contract is permanently disabled after the deadline. After the deadline, only the `AdminWithdraw` path is available, and any unreleased cMATRA returns to the project treasury under [README → Deadline and After-Window Handling](README.md#deadline-and-after-window-handling).
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

**Version:** 1.0 (draft) | **Date:** May 2026 | **Status:** Pre-launch placeholder — see TBD markers above\
**Owner:** Flux Point Studios. This page is updated as launch milestones land.
