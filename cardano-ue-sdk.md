---
description: Unreal Engine SDK (CORE)
---

# cardano-ue-sdk

## Flux Point Cardano SDK — Core (FAB)

> **SHARDS is no longer part of this SDK.** The free UE 5.5 build shipped with a demo integration that used the legacy SHARDS token. SHARDS was retired in the [cMATRA Token Merger](materios/cmatra-token-merger/README.md), and that dependency is being removed. The licensed Core SDK runs on ADA only and needs no Flux Point token. SHARDS holders should redeem before **November 28, 2026**.

### Setup

* Install plugin into your project Plugins folder.
* Project Settings > CardanoSdk:
  * CurrentNetwork: mainnet or preprod
  * ApiKey: Blockfrost project\_id
  * BaseUrl: https://cardano-mainnet.blockfrost.io/api/v0 or preprod URL
  * TreasuryAddress: preconfigured (locked in FAB)
  * PlatformFeeBps: preconfigured (locked in FAB)
  * PlatformMinLovelace: 2,000,000 (locked in FAB)

### Blueprint Quickstart

* Use `ImportMnemonic` once to load your funded 12-word wallet.
* Toggle `ForceImport` true once to overwrite any saved wallet, then false.
* Flow: BeginPlay → Branch(ForceImport)
  * True: Delete Game in Slot → ImportMnemonic → LogIn
  * False: Has Wallet → (if false ImportMnemonic) → LogIn
* After `LogIn`: `GetAddress` → `GetBalance` → `Mint Asset`.

### Mint Configuration

* CIP-25 struct allows setting MintPriceLovelace and optional CreatorAddress.
* Platform fee: min(percentage of price, percentage of network fee), floored at 2 ADA to treasury.
* Creator payout: (MintPriceLovelace - platform\_fee) to CreatorAddress when provided.

### Reliability & Logging

* Detailed logs on all nodes (errors include Cardano C messages).
* Validity window set late and auto-repaired if encoding drifts.
* HTTP submit fallback to Blockfrost with application/cbor.

### Notes

* Requires Blockfrost key.
* Native script uses after-slot timelock.
* Ships with cardano-c runtime libs under ThirdParty.

### Support

* Docs: https://docs.fluxpointstudios.com/cardano-ue-sdk
* Discord: https://discord.gg/fluxpointstudios
