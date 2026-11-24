---
description: The next generation of the Sovereign Book on Cardano preview. Its launch, sales and settlements, the FluidTokens, Liqwid and Babel-fee integrations, each linked to its explorer page, and what remains open.
---

# Preview Proof

Every transaction below landed on Cardano preview, with test ADA and test tokens. The next generation
of the Book launched there on 2026-11-18. The sales landed on 2026-11-18 to 2026-11-20, the
settlements on 2026-11-21 and 2026-11-22, and a governed table change on 2026-11-23. Each hash opens
that transaction on Cexplorer.

The generation runs two instances. The ADA instance sells cover paid in tADA. The USDCx instance
sells cover paid in preview USDCx, and its offers can also hold Liqwid qUSDCx receipts as backing.
Prices are in USD.

## What is proven and what is open

| Requirement | State |
|-------------|-------|
| Both instances launched, with every applied script hash rebuilt and matched to its reference script on chain | Proven |
| Sale on the ADA instance at the 20 tADA minimum premium | Proven, four times |
| Sale on the USDCx instance at the 10 USDCx minimum premium | Proven, five times |
| Treasury share paid in the transaction's treasury donation field | Proven on every sale that runs alone |
| Buyer cancel with a 90% premium refund | Proven, three times |
| Offer owner cancel | Proven, once |
| A sale whose leg is backed partly by Liqwid qUSDCx receipts | Proven, five times |
| A sale paid entirely in USDCx, with no net change to the buyer's ADA | Proven, twice, the second spending the tank the first one left |
| A FluidTokens loan and its cover opened in one transaction | Proven, once |
| Cover on a FluidTokens liquidation event, paid by a Claim after the protocol's own liquidation | Proven, once: 30 tADA paid |
| Expire of a cover whose term ended with no event | Proven, once: 25 tADA returned |
| A row added to the USDCx instance's asset table through proposal and enact | Proven, once |
| Claim on a price feed | Open. No price Claim has run on this generation |
| Claim paid in qUSDCx receipts | Open |
| Expire of the open sales | Due from 2026-11-27, see [The sales](#the-sales) |
| Cover sold on the second event slot | Scheduled: a loan bound to the slot, a sale on it, the loan's liquidation and the Claim |

## Launch

The launch spent 16 transactions in 10 minutes. The observer and accrual scripts are shared by both
instances.

| Step | Tx | Height | Detail |
|------|----|--------|--------|
| Observer reference script | [9de2148efb27ee18e4c169566860c92928dfcf097c1349549ad74c4d810fc222](https://preview.cexplorer.io/tx/9de2148efb27ee18e4c169566860c92928dfcf097c1349549ad74c4d810fc222) | 4869978 | Shared |
| Accrual reference script | [0b3f28848b6b718b083b44f1cc8f7b95fe196e9c484f44c010bd22a3a89f0c20](https://preview.cexplorer.io/tx/0b3f28848b6b718b083b44f1cc8f7b95fe196e9c484f44c010bd22a3a89f0c20) | 4869980 | Shared |
| ADA table | [6a659cf8f39f4ac59a254869a2bf081c55d36b803e02af12b54b886906925366](https://preview.cexplorer.io/tx/6a659cf8f39f4ac59a254869a2bf081c55d36b803e02af12b54b886906925366) | 4869981 | Mints the table token |
| ADA lane Heads | [9b9df872bf82aeaf2f03a9c297fa44450cff0499a48f3b8345aaae1c97dd08ae](https://preview.cexplorer.io/tx/9b9df872bf82aeaf2f03a9c297fa44450cff0499a48f3b8345aaae1c97dd08ae) | 4869982 | Mints the 4 lane Heads |
| ADA position reference script | [03866d9beb09014515428a76f9e4c05539552b706194feb70084c739d8461ea9](https://preview.cexplorer.io/tx/03866d9beb09014515428a76f9e4c05539552b706194feb70084c739d8461ea9) | 4869983 | 14,220 bytes |
| ADA book reference script | [3ad41ef9aad2352212c0e7cdf0f69c187eab8a508675a549ae78c9707280849c](https://preview.cexplorer.io/tx/3ad41ef9aad2352212c0e7cdf0f69c187eab8a508675a549ae78c9707280849c) | 4869984 | |
| ADA table reference script | [ed577bdb8109d08b3ac048241bd804bee9e8e84e1f3f5245cfc339ca279f8600](https://preview.cexplorer.io/tx/ed577bdb8109d08b3ac048241bd804bee9e8e84e1f3f5245cfc339ca279f8600) | 4869987 | |
| USDCx table | [54ffb1837e859e54a6cb40ed22f518b885385ec00ab3eb921cfd5a83195f1697](https://preview.cexplorer.io/tx/54ffb1837e859e54a6cb40ed22f518b885385ec00ab3eb921cfd5a83195f1697) | 4869989 | Mints the table token |
| USDCx lane Heads | [3be298838d79087796fcd0df8e5b9c3e2054da864e4e15bbfd9fc0f3b4843652](https://preview.cexplorer.io/tx/3be298838d79087796fcd0df8e5b9c3e2054da864e4e15bbfd9fc0f3b4843652) | 4869990 | Mints the 4 lane Heads |
| USDCx position reference script | [57463c1df54b4bc6ee4d3b81d611545bdc31c41a9ed8be854f170e8159eee9c4](https://preview.cexplorer.io/tx/57463c1df54b4bc6ee4d3b81d611545bdc31c41a9ed8be854f170e8159eee9c4) | 4869993 | 14,331 bytes |
| USDCx book reference script | [ae24890fcd3721341dd93b0dc66e74610070c147fc3773b33647a443fad3472d](https://preview.cexplorer.io/tx/ae24890fcd3721341dd93b0dc66e74610070c147fc3773b33647a443fad3472d) | 4869994 | |
| USDCx table reference script | [bcb0f8aeadd54638dd349ba7917ced75aaaa7c8cb405b88404527c209b0a4de6](https://preview.cexplorer.io/tx/bcb0f8aeadd54638dd349ba7917ced75aaaa7c8cb405b88404527c209b0a4de6) | 4869997 | |
| Liqwid observer reference script | [917bece2234dc6b0fdc60243372b80772552c91e21860c68f45fc1aac994c383](https://preview.cexplorer.io/tx/917bece2234dc6b0fdc60243372b80772552c91e21860c68f45fc1aac994c383) | 4869998 | USDCx instance only |
| Reference sheets | [bc72910169088cc81aa1af1197141f04b745537cc23dd4f21d1af2e3734e2abc](https://preview.cexplorer.io/tx/bc72910169088cc81aa1af1197141f04b745537cc23dd4f21d1af2e3734e2abc) | 4870002 | |
| | [3ed87d346c5961718088f0ef12ea2a036a99c994521c23411f24a136041cd17b](https://preview.cexplorer.io/tx/3ed87d346c5961718088f0ef12ea2a036a99c994521c23411f24a136041cd17b) | 4870003 | |
| | [b3a56f4956935f674439b742700a871a469b5d7cbaaf7176c26659cd88f17ae3](https://preview.cexplorer.io/tx/b3a56f4956935f674439b742700a871a469b5d7cbaaf7176c26659cd88f17ae3) | 4870004 | The event-slot sheet |

## Scripts

| Script | ADA instance | USDCx instance |
|--------|--------------|----------------|
| Position | [`60e90693…`](https://preview.cexplorer.io/script/60e9069318a5a5f6adcd8be58a04a616e50fe14f7359d82239843537) | [`2c7e04ff…`](https://preview.cexplorer.io/script/2c7e04ff848e684df95ce990f0becad0e891eea4b0e5d9e5b1610534) |
| Book | [`5027f065…`](https://preview.cexplorer.io/script/5027f0652fc707ea4fdba2bbc6d312beb6ed3b15ca65a9e7d625ccb5) | [`b52e2dbc…`](https://preview.cexplorer.io/script/b52e2dbcbccf5883431167c8af0abc85356db3d3e0adc31a00d6c25b) |
| Asset table | [`f07f33e3…`](https://preview.cexplorer.io/script/f07f33e36027037190ab528c3e03cbcabc0a3c4a6419355edcdaa3a9) | [`99f4c222…`](https://preview.cexplorer.io/script/99f4c22238e4c6ec481ba2f364dd2e72612a9857f54fd7c5a68f8d4f) |
| Lane Heads | [`c1e13248…`](https://preview.cexplorer.io/policy/c1e132480c38cf39c6e0244d7ae81b4dd04cad81d96129da946ddf84) | [`b3083a9d…`](https://preview.cexplorer.io/policy/b3083a9db61f5d86136927d9d1f958fd452edc5bc22783cb07ef4fa7) |
| Observer | [`8f5b93ae…`](https://preview.cexplorer.io/script/8f5b93aefb8f471f19128ab92e56d04b2435e9f88bcdfd35238211c8), shared | |
| Accrual | [`0457b398…`](https://preview.cexplorer.io/script/0457b39838e82e70bb0672860face954ce1c8c99d71591316ae43f30), shared | |
| Liqwid observer | | [`b94e42c5…`](https://preview.cexplorer.io/script/b94e42c56ad25c04a0a047d4f04e257193fc8a3914495a6de75da926) |
| Babel-fee tank | | [`d850e8a3…`](https://preview.cexplorer.io/script/d850e8a33e53336089854dca728a8a3d205faa35a3c407bcd9d9cbad), with its oracle [`2c31ba00…`](https://preview.cexplorer.io/script/2c31ba008de474f84c0f3da5063e1d02048a0f0de3aef79791b4778d) |

## The sales

| Sale | Tx | Height | Instance | Feed | Coverage | Premium | Strike | Start (UTC) | Expiry (UTC) | Outcome |
|------|----|--------|----------|------|----------|---------|--------|-------------|--------------|---------|
| 1 | [cf447ae7bebb43c66ceb07644a7e8db142e9baf901529d13e2dd5ff436522e67](https://preview.cexplorer.io/tx/cf447ae7bebb43c66ceb07644a7e8db142e9baf901529d13e2dd5ff436522e67) | 4870491 | ADA | ADA/USD | 50 tADA | 20 tADA | 0.104 | 2026-11-18 22:09:03 | 2026-11-25 22:09:03 | Open |
| 2 | [0c522553a6b4259de39f2b54ccbee6e119cb4740447ab4df473b93617f284faf](https://preview.cexplorer.io/tx/0c522553a6b4259de39f2b54ccbee6e119cb4740447ab4df473b93617f284faf) | 4871054 | ADA | ADA/USD | 25 tADA | 20 tADA | 0.103979 | 2026-11-19 02:17:35 | 2026-11-26 02:17:35 | Open. Sold with a FluidTokens loan |
| 3 | [3f365d6b8aa9ce90b547bf1ad503fdb35226b6adec305e369f2aaed6354fb273](https://preview.cexplorer.io/tx/3f365d6b8aa9ce90b547bf1ad503fdb35226b6adec305e369f2aaed6354fb273) | 4872938 | USDCx | ADA/USD | 50 USDCx | 10 USDCx | 0.104 | 2026-11-19 16:22:13 | 2026-12-04 16:22:13 | Buyer cancel |
| 4 | [140a27f9f3d4af8354629d8116971f60fb537c7c11219e6f9051632a1cb1747f](https://preview.cexplorer.io/tx/140a27f9f3d4af8354629d8116971f60fb537c7c11219e6f9051632a1cb1747f) | 4873275 | ADA | FluidTokens event | 25 tADA | 20 tADA | 0.5 | 2026-11-19 18:49:46 | 2026-11-20 18:49:46 | Expired |
| 5 | [44c3c22bd42dcb865499f8ca6abf9e98d395e7819a2e55fa1a1ac9e7acb10f86](https://preview.cexplorer.io/tx/44c3c22bd42dcb865499f8ca6abf9e98d395e7819a2e55fa1a1ac9e7acb10f86) | 4873349 | ADA | FluidTokens event | 30 tADA | 20 tADA | 0.5 | 2026-11-19 19:19:39 | 2026-11-22 19:19:39 | Claimed |
| 6 | [d3f8902dbc194ad1c6e338f6d086bffd28e8741f99ea1572b684cdd02f3304a2](https://preview.cexplorer.io/tx/d3f8902dbc194ad1c6e338f6d086bffd28e8741f99ea1572b684cdd02f3304a2) | 4873593 | USDCx | ADA/USD | 30 USDCx | 10 USDCx | 0.11 | 2026-11-19 21:01:31 | 2026-11-26 21:01:31 | Buyer cancel |
| 7 | [93ddcf57a64ed201db564cd6cc0bde5f033d38402d5f2c36521bd80a64449dcb](https://preview.cexplorer.io/tx/93ddcf57a64ed201db564cd6cc0bde5f033d38402d5f2c36521bd80a64449dcb) | 4873797 | USDCx | ADA/USD | 600 USDCx | 16.5 USDCx | 0.0987 | 2026-11-19 22:24:33 | 2026-11-26 22:24:33 | Open |
| 8 | [5c368c492c31cf7d88f85fd27cff6666e857299781d11998993332f7d44f244b](https://preview.cexplorer.io/tx/5c368c492c31cf7d88f85fd27cff6666e857299781d11998993332f7d44f244b) | 4874443 | USDCx | ADA/USD | 36 USDCx | 10 USDCx | 0.125 | 2026-11-20 02:59:41 | 2026-11-27 02:59:41 | Open. Paid in USDCx only |
| 9 | [8619d1a5fd55a65e54855f4d593732cc5a6912984fe9081aa8b8523d9ea5bf2c](https://preview.cexplorer.io/tx/8619d1a5fd55a65e54855f4d593732cc5a6912984fe9081aa8b8523d9ea5bf2c) | 4874556 | USDCx | USDT/USD | 50 USDCx | 10 USDCx | 0.90 | 2026-11-20 03:40:17 | 2026-11-27 03:40:17 | Buyer cancel |
| 10 | [ec59f865247380193e18f097a73fe33840d8369f9a992d3572437ce795bd1b4f](https://preview.cexplorer.io/tx/ec59f865247380193e18f097a73fe33840d8369f9a992d3572437ce795bd1b4f) | 4874853 | USDCx | ADA/USD | 36 USDCx | 10 USDCx | 0.125 | 2026-11-20 05:38:13 | 2026-11-27 05:38:13 | Open. Paid in USDCx only |

Each premium is the instance's minimum except sale 7's 16.5 USDCx. These are test sheets chosen to
exercise the path, and they are no guide to market pricing. An open sale can be expired 48 hours
after its expiry, so sale 1 is the first due, from 2026-11-27 22:09:03.

Every sale except sale 2 carries the relay's mark at metadata label 1986. Sale 2 was built together
with the loan beside it and carries none. The validators do not read the mark.

**Fees and the Treasury share.** Each ADA-instance sale pays a fee output of 1.13784 tADA to the
protocol address, which is the ledger's minimum for that output and above the 2% rule, and 0.1 tADA
in the transaction's treasury donation field, which is 0.5% of the 20 tADA premium. Each
USDCx-instance sale pays 2% of the premium in USDCx to the protocol address, and the 0.5% share in
ADA in the treasury donation field: 0.336418 tADA on sale 8, for example. Sale 2 shares its
transaction with FluidTokens' scripts, so its donation field is empty and the share goes to the
accrual script as a 1.01716 tADA output instead.

**Buyer cancels.** Sales 3, 6 and 9 were cancelled by their buyer within minutes. Each cancel
refunded 9 USDCx, 90% of the 10 USDCx premium, burned the position's markers and returned the leg to
its underwriter: [ad2f5100c8203639e66cc1464c9ba3584af2673738647431c15204d19a79e608](https://preview.cexplorer.io/tx/ad2f5100c8203639e66cc1464c9ba3584af2673738647431c15204d19a79e608)
(height 4872956),
[36acd2bb39c9aa23f8814532470099040a6cedb692f11e87d74a7a2611079e1d](https://preview.cexplorer.io/tx/36acd2bb39c9aa23f8814532470099040a6cedb692f11e87d74a7a2611079e1d)
(height 4873653) and
[2196ecb1da5e7dccb860e237c00eddbbc4deebd4dc37e376b6855737ff01fad5](https://preview.cexplorer.io/tx/2196ecb1da5e7dccb860e237c00eddbbc4deebd4dc37e376b6855737ff01fad5)
(height 4874559). The legs of sales 3 and 6 held qUSDCx receipts, and both went back with them.

**Owner cancel.** The owner of offer
[44233563e310312544be0e7de521c02524a1221d2510c3c0b5961b407b5f8f74](https://preview.cexplorer.io/tx/44233563e310312544be0e7de521c02524a1221d2510c3c0b5961b407b5f8f74)
(height 4874048: 15 tADA, 24 USDCx and 400,000,000 qUSDCx) cancelled it in
[34fae59e6f92e3e14419384d148a2bdcb7779fa908e9eefc880e0d7d35ea69ce](https://preview.cexplorer.io/tx/34fae59e6f92e3e14419384d148a2bdcb7779fa908e9eefc880e0d7d35ea69ce)
(height 4874599). Its whole contents went to the owner's key under the offer's own stake credential,
less the 0.273097 tADA network fee.

## Liqwid receipts as backing

An underwriter supplied 20,000 USDCx to Liqwid's preview USDCx market in
[09170c15ec22494c18be62892e99b62dfa6df3aaff37b816489549cc67a81a4f](https://preview.cexplorer.io/tx/09170c15ec22494c18be62892e99b62dfa6df3aaff37b816489549cc67a81a4f)
(height 4873473, 2026-11-19 20:10:51) and received 999,664,081,211 qUSDCx receipts. It then posted
[7f00ad15a6ff0fcaaee5f3b051d5e82e5adc53c7a5c2ee38d98545fe163f1d45](https://preview.cexplorer.io/tx/7f00ad15a6ff0fcaaee5f3b051d5e82e5adc53c7a5c2ee38d98545fe163f1d45)
(height 4873761), an offer of 600 USDCx and 9,900,000,000 qUSDCx on the USDCx instance.

Sale 7 drew on that offer. Its leg holds 450 USDCx and 8,622,102,701 qUSDCx for 600 USDCx of
coverage. The receipts cover the other 150 USDCx and are worth 172.5 USDCx at the market's exchange
rate, the same rate the supply was minted at: 150 USDCx plus a 15% haircut. Sales 3, 6, 8 and 10 have
receipt-backed legs on the same terms. The supplied USDCx stays in the Liqwid market while its
receipts back the cover.

## Paying in USDCx with no ADA

A Babel-fee tank lets a buyer pay every ADA part of a sale in USDCx. The tank is our deployment of
FluidTokens' Aquarium validators. Its oracle takes a signed USDCx price in lovelace. It was deployed in
[7628c11170854b73d4b6ce827689425defb80041c2cbc1e7c6e08415efc1ef8c](https://preview.cexplorer.io/tx/7628c11170854b73d4b6ce827689425defb80041c2cbc1e7c6e08415efc1ef8c)
(height 4873602, 2026-11-19 21:00:44) with 30 tADA, its parameters token, its oracle token and two
reference scripts.

Sale 8 spent the tank. The tank paid 6.506674 tADA: the network fee, the position's deposit, the
minimum ADA of the fee output and of the tank owner's output, and the Treasury share. The buyer paid
11.001393 USDCx: the 10 USDCx premium, the 0.2 USDCx protocol fee and 0.801393 USDCx to the tank's
owner for the ADA. The buyer's ADA went in as 1.5 tADA and came out as 1.5 tADA. A separate 5 tADA
collateral input was named and not spent. The tank continued at 23.493326 tADA in output 0.

Sale 10 spent that continued output, with a different buyer. The tank paid 6.507906 tADA and the
buyer paid 10.999745 USDCx. The buyer's ADA in and out were both 27.075285 tADA.

## FluidTokens liquidation cover

A FluidTokens V4 loan can be bound to an event slot on the asset table. The publisher signs the
slot's reading: 1.0 while the bound loan is open and 0.1 once FluidTokens' own liquidation of that
loan lands. A cover on the slot has a strike of 0.5, so it pays when the loan is liquidated during the
term. A late repayment does not trigger it.

| Step | Tx | Height | Time (UTC) | Detail |
|------|----|--------|------------|--------|
| Loan | [4a08d2a86f05b769031d21e7b2a767739199b1c6b3a739f43a04d9045f0aa516](https://preview.cexplorer.io/tx/4a08d2a86f05b769031d21e7b2a767739199b1c6b3a739f43a04d9045f0aa516) | 4872975 | 2026-11-19 16:31:13 | 60 tADA lent against 150 tADA of collateral |
| Cover, 1 day | [140a27f9f3d4af8354629d8116971f60fb537c7c11219e6f9051632a1cb1747f](https://preview.cexplorer.io/tx/140a27f9f3d4af8354629d8116971f60fb537c7c11219e6f9051632a1cb1747f) | 4873275 | 2026-11-19 18:46:16 | Sale 4, 25 tADA |
| Cover, 3 days | [44c3c22bd42dcb865499f8ca6abf9e98d395e7819a2e55fa1a1ac9e7acb10f86](https://preview.cexplorer.io/tx/44c3c22bd42dcb865499f8ca6abf9e98d395e7819a2e55fa1a1ac9e7acb10f86) | 4873349 | 2026-11-19 19:16:14 | Sale 5, 30 tADA |
| Liquidation | [8e5b30c82102d860caf10091bf6ed081dc61fd6a2e662071586b63782ea6b8bb](https://preview.cexplorer.io/tx/8e5b30c82102d860caf10091bf6ed081dc61fd6a2e662071586b63782ea6b8bb) | 4877777 | 2026-11-21 02:01:44 | The lender takes the 150 tADA collateral and burns the loan token |
| Event reading | [072bfcd05032084c772ef6eb9988d4f9dfced76a1dd979fbdd0d80dd5e692c0c](https://preview.cexplorer.io/tx/072bfcd05032084c772ef6eb9988d4f9dfced76a1dd979fbdd0d80dd5e692c0c) | 4877961 | 2026-11-21 03:16:59 | Output 4: the slot reads 0.1, with the event time 02:01:44 |
| Claim of sale 5 | [79ee8fa669fc3e1bbd98bf7d8dfab9c96f1979a8f380b2e68a5ba6b988d4ac31](https://preview.cexplorer.io/tx/79ee8fa669fc3e1bbd98bf7d8dfab9c96f1979a8f380b2e68a5ba6b988d4ac31) | 4877963 | 2026-11-21 03:17:50 | 30 tADA paid to the borrower |
| Expire of sale 4 | [56e5de4ff58edc9da867a20eff2b4ab6c47c68930e3f91d613c96158b23ffd46](https://preview.cexplorer.io/tx/56e5de4ff58edc9da867a20eff2b4ab6c47c68930e3f91d613c96158b23ffd46) | 4883399 | 2026-11-22 18:52:55 | 25 tADA back to the underwriter |

**The Claim.** The borrower bought two covers on one loan. The 1-day cover ended on 2026-11-20 at
18:49:46 with the loan still open. The lender liquidated the loan through FluidTokens on 2026-11-21 at
02:01:44, inside the 3-day cover's term. The publisher's reading at 03:16:59 named the loan in its
metadata and set the slot to 0.1. The Claim landed 51 seconds after that reading, paid the full
30 tADA coverage and burned the position's markers. Since the sale committed no payout address, the
payout went to the insured key's address with no stake part. A sale can commit a payout address, and
the contract then pays exactly that address.

**The Expire.** The 1-day cover was expired 48 hours after its term ended, with no event in it. The Expire
returned the 25 tADA leg to its underwriter under the same stake credential and burned the markers.

The Claim and the Expire were each submitted by a wallet that was neither the insured nor the
underwriter. Neither transaction needed a signature from either of them.

**Loan and cover in one transaction.** Sale 2 opened a FluidTokens loan of 25 tADA against 50 tADA
of collateral and sold 25 tADA of ADA/USD Barrier cover to the same borrower, in one transaction
with 10 script executions.

**A liquidation with no cover.** A second loan,
[2999f27c61bfeb102d97ab120ef633ce53d1e9a59689301440efc3cd412da888](https://preview.cexplorer.io/tx/2999f27c61bfeb102d97ab120ef633ce53d1e9a59689301440efc3cd412da888)
(height 4871072), lent 60 tADA against 150 tADA and had no cover. Its lender liquidated it in
[4373e4d8ed6ab04c0c40040ef10816c9feb1c7e9c77aa7dc7ee31e16e98c285d](https://preview.cexplorer.io/tx/4373e4d8ed6ab04c0c40040ef10816c9feb1c7e9c77aa7dc7ee31e16e98c285d)
(height 4875747, 2026-11-20 11:58:14) and recovered the 150 tADA collateral.

## A governed table change

A second event slot was added to the USDCx instance's asset table on 2026-11-23.

| Step | Tx | Height | Time (UTC) | Detail |
|------|----|--------|------------|--------|
| Slot token | [7bdfe1a192bf287826cb2d11a247a052e95564f666b0e461ee04c662b00e44d2](https://preview.cexplorer.io/tx/7bdfe1a192bf287826cb2d11a247a052e95564f666b0e461ee04c662b00e44d2) | 4886843 | 20:16:22 | Mints the slot's one token under a native policy that can no longer mint after slot 128809557 |
| Proposal | [8d24f0f901abc5f0544cc4b5b9ef638573f1b0bbcdf867e69ed4d72a9af47475](https://preview.cexplorer.io/tx/8d24f0f901abc5f0544cc4b5b9ef638573f1b0bbcdf867e69ed4d72a9af47475) | 4886877 | 20:33:35 | Table epoch 0 with two rows. Pending: three rows, effective after 21:06:28 |
| Enact | [03dc7ed150fe778fc8b4c38e85a16f3eb60730054c4b43fd25cd2707b5e071ba](https://preview.cexplorer.io/tx/03dc7ed150fe778fc8b4c38e85a16f3eb60730054c4b43fd25cd2707b5e071ba) | 4886956 | 21:08:36 | Table epoch 1 with three rows, nothing pending |
| Offer on the new row | [506f60968c4518e544538c3da12fb16815630b3aa1c3001d44a550afb19d7c6a](https://preview.cexplorer.io/tx/506f60968c4518e544538c3da12fb16815630b3aa1c3001d44a550afb19d7c6a) | 4886960 | 21:10:05 | 15 tADA, 4 USDCx and 1,000,000,000 qUSDCx on the new slot's feed |

The new row is an event row that accepts qUSDCx receipts as backing. No new generation was needed to
add it. A loan bound to the slot, a sale on it, the loan's liquidation and the Claim are scheduled
next.

## What this does not prove

This is preview, with test ADA, test tokens and an unaudited generation, which is audited at its code
freeze. One Claim has run, on a FluidTokens liquidation event. No price feed has triggered a Claim on
this generation, and no Claim has paid in qUSDCx receipts. The sales still open can be
settled from 2026-11-27. The Babel-fee tank is our deployment of FluidTokens' Aquarium validators with its own
oracle, and that combination has had no independent review.
