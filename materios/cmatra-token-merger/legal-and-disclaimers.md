---
description: >-
  Legal disclaimers, risk disclosures, jurisdiction notes, and tax / KYC
  placeholders for the cMATRA Token Merger. Read this before participating in
  the redemption window.
---

# Legal & Disclaimers

This page sets the legal context for participating in the cMATRA Token Merger. Read it before you redeem. The counsel-approved disclosures rendered on the live [redemption portal](https://fluxpointstudios.com/matra-merger) and the FPS [Terms of Service](https://fluxpointstudios.com/tos) are the canonical form; if anything here conflicts with the portal disclosures or the ToS, the portal and the ToS control.

***

## Not Investment Advice

cMATRA and MATRA are **utility tokens**.

* Nothing on this page — or anywhere else in the cMATRA docs, the FPS Discord, the litepaper, or the rate table — constitutes investment advice, financial advice, legal advice, or tax advice.
* Nothing here constitutes an offer to sell, or a solicitation of an offer to buy, a security in any jurisdiction.
* No statement on this site should be read as a representation about future price, return, market cap, or liquidity. Markets are markets; we do not control them.
* Materios and Flux Point Studios make no recommendation about whether any specific person should or should not redeem.

If you are unsure whether the merger is appropriate for your circumstances, consult an independent professional advisor before participating.

***

## Risk Disclosure

Participating in the cMATRA merger carries real risk. The following list is not exhaustive — it is the floor, not the ceiling.

### Smart-contract risk

* The Cardano-side merger contracts and the Materios chain are software. Software has bugs.
* Audits, peer review, and a public testnet (`materios_preprod_v6`) reduce — but do not eliminate — the chance of a critical bug surfacing during or after launch.
* A bug in the surrender script, the cMATRA minting script, the bridge contracts, or the Materios runtime could result in **loss of funds, frozen funds, or incorrect minting**.
* See [Audit Status](launch-readiness.md#audit-status) for the current security review posture.

### Market risk

* cMATRA and MATRA will be traded on at least one CLOB and on AMM secondary venues. Trading is voluntary and subject to ordinary market forces.
* **Price can go to zero.** No protocol mechanism guarantees a floor.
* **Liquidity can disappear.** Maker rebates, Protocol-Owned Liquidity, and other Liquidity sub-bucket programs are designed to keep depth healthy, but none of them are price guarantees. If maker quotes are pulled, spreads widen and slippage rises.
* **Volatility is normal at launch.** Initial price discovery on a new token routinely produces 30%+ daily swings. Do not redeem capital you cannot afford to see at half price.

### Counterparty and operational risk

* The merger is operated by Flux Point Studios under the Materios protocol. The on-chain validators, the publishing scripts, and the launch package are operated by humans.
* **Admin keys exist.** A multisig-controlled administrator key holds permissions over the merger contracts (e.g. closing the surrender path at the deadline, withdrawing unreleased cMATRA from the public pool after expiry). Key compromise, key holder coercion, or operator error are non-zero risks.
* **Bridges are not yet operational.** cMATRA at launch is a Cardano-native asset. The Materios bridge — which moves cMATRA into MATRA on the Materios chain — is future work. See [FAQ Q37](faq.md#37-do-i-need-to-bridge-to-materios-immediately-after-redeeming). When the bridge ships, it introduces its own counterparty surface (validator signers, peg reserve solvency).
* **Wallet and custodian risk.** If your Cardano wallet is compromised, or if you delegate custody to a third party (exchange, custodian, contract), you can lose access to your cMATRA. Materios cannot recover lost or stolen tokens.

### Regulatory risk

* Crypto regulation is in active flux in every major jurisdiction.
* The legal classification of cMATRA and MATRA in any specific jurisdiction is a question for that jurisdiction's authorities and your own counsel — not for these docs.
* **Future regulation may restrict redemption, transfer, or use** of cMATRA or MATRA in your jurisdiction even after you redeem. Materios cannot indemnify users against regulatory action.
* See [Jurisdiction Status](#jurisdiction-status) below.

### Protocol-evolution risk

* Materios is a live protocol. The runtime is upgraded via governance; pallets, fees, and reserves evolve.
* Documented governance-approved corrections to the rate table are explicitly contemplated by [Window mechanics](README.md#window-mechanics) — they are not promised to never happen.
* Future runtime upgrades, hard forks, or migration events may change how cMATRA behaves on-chain.

***

## Jurisdiction Status

The cMATRA Merger Portal is operated by **Flux Point Studios, Inc.**, a Delaware corporation. The Portal is **not offered to, and may not be used by**, any person located in, organized under the laws of, or ordinarily resident in any jurisdiction subject to comprehensive sanctions by the U.S. Department of the Treasury's Office of Foreign Assets Control — currently including **Cuba, Iran, North Korea, Syria**, and the **Crimea, Donetsk, and Luhansk** regions of Ukraine — nor by any person identified on any U.S., U.K., E.U., or U.N. sanctions or denied-persons list. By connecting a wallet you represent that you are not such a person and are not acting on behalf of such a person.

You are solely responsible for compliance with the laws, regulations, and tax rules of your jurisdiction. Materios reserves the right to geoblock the redemption frontend in any jurisdiction where it cannot comply with local law.

The full counsel-approved restricted-persons clause, dispute-resolution terms (Utah governing law, AAA binding individual arbitration, class-action and jury-trial waiver, 30-day opt-out), and corporate-law venue are rendered on the live [redemption portal](https://fluxpointstudios.com/matra-merger) and in the [Terms of Service](https://fluxpointstudios.com/tos).

***

## KYC / AML

* Public redemptions proceed via on-chain CIP-30 wallet signature. **No general KYC is collected** at the Portal.
* Strategic-allocation participants are subject to a separate enhanced-due-diligence process and are contacted directly.
* Materios reserves the right to delay or refuse any redemption that triggers sanctions, anti-money-laundering, or counter-terrorist-financing review until that review is complete.

***

## Tax

**Token redemption may be a taxable event in your jurisdiction.**

* No part of this site is tax advice.
* Consult a qualified tax professional licensed in your jurisdiction before redeeming or trading cMATRA or MATRA.
* Materios will not file or remit tax on your behalf, and does not issue tax forms beyond what is legally required of the operating entity.
* Keep your own records: surrender transaction hashes, on-chain timestamps, the rate at which you redeemed, and any subsequent trades. Block explorers are public but transient — back up the data you need.

***

## No Warranty — Software Provided As Is

* The Materios chain, the merger contracts, the cMATRA token, the redemption frontend, the SDKs, and any other software referenced in these docs are provided **"AS IS" and "AS AVAILABLE,"** with **no warranties of any kind**, express or implied, including without limitation any implied warranties of merchantability, fitness for a particular purpose, non-infringement, accuracy, or availability.
* Use of the protocol is at your own risk.
* To the maximum extent permitted by applicable law, Materios, Flux Point Studios, and their affiliates, contributors, validators, and operators **disclaim all liability** for any direct, indirect, incidental, special, consequential, or exemplary damages arising from your use of (or inability to use) the protocol, the merger, the redemption interface, or any related software.

***

## Force Majeure and Emergency Pause

* Materios may **pause, throttle, modify, or roll back the merger** in response to a discovered vulnerability, an active exploit, a credible threat to user funds, a regulatory directive, a chain outage, a Cardano L1 incident, or any other event that materially threatens the safe operation of the protocol.
* Any such pause will be announced through the official channels ([Discord](https://discord.gg/MfYUMnfrJM), [docs site](https://docs.fluxpointstudios.com), Materios chain governance notes) as soon as it is safe to do so.
* The duration of any pause is bounded by the underlying cause. The administrator does not have unilateral authority to permanently terminate the merger before the 6-month window expires absent one of the conditions above.

***

## Updates and Amendments

These docs are versioned in the public [`Flux-Point-Studios/docs`](https://github.com/Flux-Point-Studios/docs) repository.

* Every change is tracked in the git history (commit, author, date, diff).
* Material changes to this page or to the merger terms will be **announced on [Discord](https://discord.gg/MfYUMnfrJM)** and in the [Launch Readiness](launch-readiness.md) status board.
* Trivial corrections (typos, formatting, broken links) are made without separate announcement; the commit history is the source of truth.
* If you participated in the merger under an earlier version of these terms, that version of the terms applies to your participation as of your redemption transaction. The git history preserves prior versions for audit.

***

## Authoritative Materials

If anything on this page conflicts with the final launch package, the final launch package controls. The authoritative materials are listed in the [README's Authoritative Launch Materials section](README.md#authoritative-launch-materials).

***

**Version:** 1.0 | **Launch:** May 28, 2026\
**Owner:** Flux Point Studios.
