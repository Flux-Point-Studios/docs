---
description: Gen-3 of the Sovereign Book on Cardano preview, its launch transactions, contract hashes, standing offers and first sale, each linked to its explorer page.
---

# Gen-3 on preview

Gen-3 is the next generation of the Sovereign Book, described in
[the Overview](README.md#the-next-generation). It runs on Cardano preview with test ADA, as two
instances: one sells cover paid in ADA, the other cover paid in USDCx. The launch landed on
2026-11-18 between 18:25 and 18:35 UTC, the offers between 20:07 and 20:10 UTC, and the first sale
at 22:05:48 UTC. Each transaction and script hash opens on Cexplorer.

## What is proven and what is open

| Requirement | State |
|-------------|-------|
| Both instances deployed, with their asset tables, lanes and reference scripts | Proven: 16 launch transactions |
| Reference price sheets published | Proven |
| Standing offers on both instances | Proven: three on the ADA instance, three on the USDCx instance |
| ADA Barrier sale on the ADA instance | Proven, once |
| Fee output and Cardano Treasury share on a sale | Proven, once |
| Sale priced at the 20 ADA minimum premium | Proven, once |
| Sale on the USDCx instance | Open. No USDCx offer has been drawn on yet |
| FluidTokens loan cover sale | Open |
| Claim or Expire on gen-3 | Open. The first position's term ends on 2026-11-25 at 22:09:03 UTC |
| Claim paid in Liqwid receipts | Open |

## The two instances

| Instance | Unit | Sells | Pays claims in |
|----------|------|-------|----------------|
| ADA | `lovelace` | ADA/USD Barrier cover and FluidTokens loan cover | ADA |
| USDCx | `d8906ca5c7ba124a0407a32dab37b2c82b13b3dcd9111e42940dcea45553444378` | ADA/USD Barrier and USDT Depeg cover, priced through the ADA/USD feed | USDCx first, then receipts from Liqwid's isolated sUSDr-USDCx market on preview |

Both instances share these rules:

- A book term runs at least 1 day.
- The premium is at least 20 ADA on the ADA instance and 10 USDCx on the USDCx instance.
- A sale is refused when its total cost to the buyer reaches the coverage.
- 2% of the premium goes to the protocol and 0.5% to the Cardano Treasury.

## Launch transactions

| Step | Tx | Height | Detail |
|------|----|--------|--------|
| Observer reference script | [9de2148efb27ee18e4c169566860c92928dfcf097c1349549ad74c4d810fc222](https://preview.cexplorer.io/tx/9de2148efb27ee18e4c169566860c92928dfcf097c1349549ad74c4d810fc222) | 4869978 | Shared by both instances |
| Accrual reference script | [0b3f28848b6b718b083b44f1cc8f7b95fe196e9c484f44c010bd22a3a89f0c20](https://preview.cexplorer.io/tx/0b3f28848b6b718b083b44f1cc8f7b95fe196e9c484f44c010bd22a3a89f0c20) | 4869980 | Shared by both instances |
| ADA: table mint | [6a659cf8f39f4ac59a254869a2bf081c55d36b803e02af12b54b886906925366](https://preview.cexplorer.io/tx/6a659cf8f39f4ac59a254869a2bf081c55d36b803e02af12b54b886906925366) | 4869981 | Asset table with ADA/USD, USDC, USDT and the FluidTokens loan event slot |
| ADA: lanes mint | [9b9df872bf82aeaf2f03a9c297fa44450cff0499a48f3b8345aaae1c97dd08ae](https://preview.cexplorer.io/tx/9b9df872bf82aeaf2f03a9c297fa44450cff0499a48f3b8345aaae1c97dd08ae) | 4869982 | Mints the 4 lane Heads |
| ADA: position reference script | [03866d9beb09014515428a76f9e4c05539552b706194feb70084c739d8461ea9](https://preview.cexplorer.io/tx/03866d9beb09014515428a76f9e4c05539552b706194feb70084c739d8461ea9) | 4869983 | |
| ADA: book reference script | [3ad41ef9aad2352212c0e7cdf0f69c187eab8a508675a549ae78c9707280849c](https://preview.cexplorer.io/tx/3ad41ef9aad2352212c0e7cdf0f69c187eab8a508675a549ae78c9707280849c) | 4869984 | |
| ADA: table reference script | [ed577bdb8109d08b3ac048241bd804bee9e8e84e1f3f5245cfc339ca279f8600](https://preview.cexplorer.io/tx/ed577bdb8109d08b3ac048241bd804bee9e8e84e1f3f5245cfc339ca279f8600) | 4869987 | |
| USDCx: table mint | [54ffb1837e859e54a6cb40ed22f518b885385ec00ab3eb921cfd5a83195f1697](https://preview.cexplorer.io/tx/54ffb1837e859e54a6cb40ed22f518b885385ec00ab3eb921cfd5a83195f1697) | 4869989 | Asset table with ADA/USD and USDT. The ADA/USD row lists the Liqwid receipt |
| USDCx: lanes mint | [3be298838d79087796fcd0df8e5b9c3e2054da864e4e15bbfd9fc0f3b4843652](https://preview.cexplorer.io/tx/3be298838d79087796fcd0df8e5b9c3e2054da864e4e15bbfd9fc0f3b4843652) | 4869990 | Mints the 4 lane Heads |
| USDCx: position reference script | [57463c1df54b4bc6ee4d3b81d611545bdc31c41a9ed8be854f170e8159eee9c4](https://preview.cexplorer.io/tx/57463c1df54b4bc6ee4d3b81d611545bdc31c41a9ed8be854f170e8159eee9c4) | 4869993 | |
| USDCx: book reference script | [ae24890fcd3721341dd93b0dc66e74610070c147fc3773b33647a443fad3472d](https://preview.cexplorer.io/tx/ae24890fcd3721341dd93b0dc66e74610070c147fc3773b33647a443fad3472d) | 4869994 | |
| USDCx: table reference script | [bcb0f8aeadd54638dd349ba7917ced75aaaa7c8cb405b88404527c209b0a4de6](https://preview.cexplorer.io/tx/bcb0f8aeadd54638dd349ba7917ced75aaaa7c8cb405b88404527c209b0a4de6) | 4869997 | |
| USDCx: Liqwid observer reference script | [917bece2234dc6b0fdc60243372b80772552c91e21860c68f45fc1aac994c383](https://preview.cexplorer.io/tx/917bece2234dc6b0fdc60243372b80772552c91e21860c68f45fc1aac994c383) | 4869998 | |
| Reference sheets | [bc72910169088cc81aa1af1197141f04b745537cc23dd4f21d1af2e3734e2abc](https://preview.cexplorer.io/tx/bc72910169088cc81aa1af1197141f04b745537cc23dd4f21d1af2e3734e2abc) | 4870002 | ADA Barrier sheet |
| | [3ed87d346c5961718088f0ef12ea2a036a99c994521c23411f24a136041cd17b](https://preview.cexplorer.io/tx/3ed87d346c5961718088f0ef12ea2a036a99c994521c23411f24a136041cd17b) | 4870003 | Depeg sheet |
| | [b3a56f4956935f674439b742700a871a469b5d7cbaaf7176c26659cd88f17ae3](https://preview.cexplorer.io/tx/b3a56f4956935f674439b742700a871a469b5d7cbaaf7176c26659cd88f17ae3) | 4870004 | FluidTokens loan event sheet |

## Contract hashes

The observer and accrual scripts are shared. Each instance has its own position, book and table
scripts.

| Script | Instance | Hash |
|--------|----------|------|
| Position | ADA | [60e9069318a5a5f6adcd8be58a04a616e50fe14f7359d82239843537](https://preview.cexplorer.io/script/60e9069318a5a5f6adcd8be58a04a616e50fe14f7359d82239843537) |
| | USDCx | [2c7e04ff848e684df95ce990f0becad0e891eea4b0e5d9e5b1610534](https://preview.cexplorer.io/script/2c7e04ff848e684df95ce990f0becad0e891eea4b0e5d9e5b1610534) |
| Book | ADA | [5027f0652fc707ea4fdba2bbc6d312beb6ed3b15ca65a9e7d625ccb5](https://preview.cexplorer.io/script/5027f0652fc707ea4fdba2bbc6d312beb6ed3b15ca65a9e7d625ccb5) |
| | USDCx | [b52e2dbcbccf5883431167c8af0abc85356db3d3e0adc31a00d6c25b](https://preview.cexplorer.io/script/b52e2dbcbccf5883431167c8af0abc85356db3d3e0adc31a00d6c25b) |
| Table | ADA | [f07f33e36027037190ab528c3e03cbcabc0a3c4a6419355edcdaa3a9](https://preview.cexplorer.io/script/f07f33e36027037190ab528c3e03cbcabc0a3c4a6419355edcdaa3a9) |
| | USDCx | [99f4c22238e4c6ec481ba2f364dd2e72612a9857f54fd7c5a68f8d4f](https://preview.cexplorer.io/script/99f4c22238e4c6ec481ba2f364dd2e72612a9857f54fd7c5a68f8d4f) |
| Accrual | Both | [0457b39838e82e70bb0672860face954ce1c8c99d71591316ae43f30](https://preview.cexplorer.io/script/0457b39838e82e70bb0672860face954ce1c8c99d71591316ae43f30) |
| Observer | Both | [8f5b93aefb8f471f19128ab92e56d04b2435e9f88bcdfd35238211c8](https://preview.cexplorer.io/script/8f5b93aefb8f471f19128ab92e56d04b2435e9f88bcdfd35238211c8) |
| Liqwid observer | USDCx | [b94e42c56ad25c04a0a047d4f04e257193fc8a3914495a6de75da926](https://preview.cexplorer.io/script/b94e42c56ad25c04a0a047d4f04e257193fc8a3914495a6de75da926) |
| ADA/USD feed the USDCx instance prices through | USDCx | [4bba53d510bf2af7d08e8edf57c3a49ccdb6b0406c8470eea03081ef](https://preview.cexplorer.io/script/4bba53d510bf2af7d08e8edf57c3a49ccdb6b0406c8470eea03081ef) |

## Standing offers

Six offers stand on preview, three on each instance's book. All six were posted by one underwriter,
under one stake credential. Sizes are as they stand on chain.

| Offer | Out-ref | Holds | Depth | Term |
|-------|---------|-------|-------|------|
| ADA Barrier, wide | [cf447ae7bebb43c66ceb07644a7e8db142e9baf901529d13e2dd5ff436522e67#3](https://preview.cexplorer.io/tx/cf447ae7bebb43c66ceb07644a7e8db142e9baf901529d13e2dd5ff436522e67) | 470 tADA | 15% to 50% | 7 to 45 days |
| ADA Barrier, deep | [e1259312b6e43dca71611074a54ec9f1bdd326ef40c6f9eaf6d544d4e4c88b9d#0](https://preview.cexplorer.io/tx/e1259312b6e43dca71611074a54ec9f1bdd326ef40c6f9eaf6d544d4e4c88b9d) | 400 tADA | 30% to 50% | 7 to 45 days |
| ADA Barrier, short | [88c59c1b3bb3d297d35b425b7b76f0a9a1deae07b4cc3c8fad5b0ed601b857f5#0](https://preview.cexplorer.io/tx/88c59c1b3bb3d297d35b425b7b76f0a9a1deae07b4cc3c8fad5b0ed601b857f5) | 300 tADA | 15% to 50% | 7 to 14 days |
| USDCx Barrier, backed by receipts | [0d4c19b0314cdb4724ffaddaf457f4469b2f17bd3edb2cee392679ba800997fc#0](https://preview.cexplorer.io/tx/0d4c19b0314cdb4724ffaddaf457f4469b2f17bd3edb2cee392679ba800997fc) | 4,000 USDCx and 20,000,000,000 receipt units | 15% to 50% | 7 to 45 days |
| USDCx Barrier, short | [c5261417827d16cb4f1746a214fc53aeaa08d49332c411a26e2df7e1c0df830c#0](https://preview.cexplorer.io/tx/c5261417827d16cb4f1746a214fc53aeaa08d49332c411a26e2df7e1c0df830c) | 1,000 USDCx and 5,000,000,000 receipt units | 15% to 50% | 7 to 14 days |
| USDCx USDT Depeg | [4dd5ce7bd9111def4853ba12dd745df8187e478beab26d28032ad23d014decf9#0](https://preview.cexplorer.io/tx/4dd5ce7bd9111def4853ba12dd745df8187e478beab26d28032ad23d014decf9) | 3,000 USDCx | 5% to 50% | 7 to 45 days |

The wide offer was posted as
[a3e5dadfd53efa164e44ff8584e8f0c9fc769b651db5461f8a2f3ab29d763fb8#0](https://preview.cexplorer.io/tx/a3e5dadfd53efa164e44ff8584e8f0c9fc769b651db5461f8a2f3ab29d763fb8)
with 500 tADA. The first sale drew on it, and its remainder of 470 tADA went back to the book at
the out-ref above. The three ADA Barrier offers and the two USDCx Barrier offers read the ADA/USD
feed and price from the ADA Barrier sheet. The Depeg offer reads USDT and prices from the Depeg
sheet. Each USDCx offer also holds 40 tADA alongside its tokens. Receipt units are counted in the
receipt token's base unit.

## The first sale

| Field | Value |
|-------|-------|
| Tx | [cf447ae7bebb43c66ceb07644a7e8db142e9baf901529d13e2dd5ff436522e67](https://preview.cexplorer.io/tx/cf447ae7bebb43c66ceb07644a7e8db142e9baf901529d13e2dd5ff436522e67) |
| Landed | 2026-11-18 22:05:48 UTC, height 4870491 |
| Position | Out-ref #1 |
| Instance | ADA |
| Cover | ADA/USD Barrier, strike 0.104 USD |
| Coverage | 50 tADA |
| Premium | 20 tADA |
| Term | 7 days, from 2026-11-18 22:09:03 to 2026-11-25 22:09:03 UTC |

The reading the sale checked had a spot of 0.150066 USD, observed at 22:02:23 UTC, and a trailing
low of 0.149403 USD, observed at 15:51:50 UTC on 2026-11-17. The 0.104 strike sits about 30% under
that trailing low, inside the wide offer's 15% to 50% band. The term starts at the upper bound of
the transaction's validity window.

**The premium is the book minimum.** The 20 tADA premium was set by gen-3's 20 ADA minimum, not by
the offer's price sheet. The sheet's rate for 7 days at this depth is 2.75%, which is 1.375 tADA on
50 tADA of cover, so the minimum applies.

**What the buyer paid.** The buyer's wallet paid 24.197254 tADA, exactly the quote:

| Item | tADA |
|------|------|
| Premium | 20 |
| Protocol fee | 1.13784 |
| Cardano Treasury share | 0.1 |
| Settler deposit, returned at settlement | 2.1119 |
| Network fee | 0.847514 |
| **Total** | **24.197254** |

The protocol fee is 2% of the premium, 0.4 tADA, raised to 1.13784 tADA, the ledger's minimum for
that output. The Treasury share is 0.5% of the premium and goes to the Cardano Treasury through the
transaction's own donation field. The settler deposit sits in the position's P-leg. The total is
under the 50 tADA coverage, so the sale passes the total-cost rule.

**What the underwriter's offer did.** The wide offer went from 500 to 470 tADA. The sale locked
50 tADA as cover in a C-leg and added the 20 tADA premium back to the offer's remainder. The C-leg
and the remainder keep the offer's stake credential. The sale carries the relay's mark at metadata
label 1986.

## What this does not prove

This is preview, with test ADA and an unaudited generation. One sale has run, on the ADA instance.
No USDCx sale, FluidTokens loan cover sale, Claim or Expire has run on gen-3, and no claim has paid
out in Liqwid receipts. The current generation's lifecycle on preprod is on
[Preprod Proof](preprod-proof.md).
