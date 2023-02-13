# Reserve - Mitigation contest details
- Total Prize Pool: $45,000 USDC
- [Warden guidelines for C4 mitigation reviews](https://code4rena.notion.site/Guidelines-for-Versus-mitigation-reviews-ed10fc5cfbf640bd8dcec66f38b343c4)
- Submit findings [using the C4 form](https://code4rena.com/contests/2023-02-reserve-versus-mitigation-contest/submit)
- Starts February 13, 2023 20:00 UTC
- Ends February 17, 2023 20:00 UTC

## Important note 

Each warden must submit a mitigation review for *every High and Medium finding* from the parent contest. **Incomplete mitigation reviews will not be eligible for awards.**

## Findings being mitigated

### High Risk Findings

- [H-01: Adversary can abuse a quirk of compound redemption to manipulate the underlying exchange rate and maliciously disable cToken collaterals](https://github.com/code-423n4/2023-01-reserve-findings/issues/310)
- [H-02: Basket range formula is inefficient, leading the protocol to unnecessary haircut](https://github.com/code-423n4/2023-01-reserve-findings/issues/235)

### Medium Risk Findings

- [M-01: Battery discharge mechanism doesn't work correctly for first redemption](https://github.com/code-423n4/2023-01-reserve-findings/issues/452)
- [M-02: attacker can make stakeRate to be 1 in the StRSR contract and users depositing tokens can lose funds because of the big rounding error](https://github.com/code-423n4/2023-01-reserve-findings/issues/439)
- [M-03: Baited by redemption during undercollateralization (no issuance, just transfer)](https://github.com/code-423n4/2023-01-reserve-findings/issues/416)
- [M-04: Redemptions during undercollateralization can be hot-swapped to steal all funds)](https://github.com/code-423n4/2023-01-reserve-findings/issues/399)
- [M-05: early user can call issue() and then melt() to increase basketsNeeded to supply ratio to its maximum value and then melt() won't work and contract contract features like issue() won't work](https://github.com/code-423n4/2023-01-reserve-findings/issues/384)
- [M-06: Too few rewards paid over periods in Furnace and StRSR](https://github.com/code-423n4/2023-01-reserve-findings/issues/377)
- [M-07: attacker can steal RToken holders funds by performing reentrancy attack during redeem() function token transfers](https://github.com/code-423n4/2023-01-reserve-findings/issues/347)
- [M-08: Asset.lotPrice() doesn't use the most recent price in case of oracle timeout](https://github.com/code-423n4/2023-01-reserve-findings/issues/326)
- [M-09: Withdrawals will stuck](https://github.com/code-423n4/2023-01-reserve-findings/issues/325)
- [M-10: Unsafe downcasting in issue(...) can be exploited to cause permanent DoS](https://github.com/code-423n4/2023-01-reserve-findings/issues/320)
- [M-11: Should Accrue Before Change, Loss of Rewards in case of change of settings](https://github.com/code-423n4/2023-01-reserve-findings/issues/287)
- [M-12: BackingManager: rsr is distributed across all rsr revenue destinations which is a loss for rsr stakers](https://github.com/code-423n4/2023-01-reserve-findings/issues/276)
- [M-13: attacker can prevent vesting for a very long time](https://github.com/code-423n4/2023-01-reserve-findings/issues/267)
- [M-14: Unsafe cast of uint8 datatype to int8](https://github.com/code-423n4/2023-01-reserve-findings/issues/265)
- [M-15: The Furnace#melt() is vulnerable to sandwich attacks](https://github.com/code-423n4/2023-01-reserve-findings/issues/258)
- [M-16: RToken permanently insolvent/unusable if a single collateral in the basket behaves unexpectedly](https://github.com/code-423n4/2023-01-reserve-findings/issues/254)
- [M-17: refresh() will revert on Oracle deprecation, effectively disabling part of the protocol](https://github.com/code-423n4/2023-01-reserve-findings/issues/234)
- [M-18: If name is changed then the domain seperator would be wrong.](https://github.com/code-423n4/2023-01-reserve-findings/issues/211)
- [M-19: In case that unstakingDelay is decreased, users who have previously unstaked would have to wait more than unstakingDelay for new unstakes](https://github.com/code-423n4/2023-01-reserve-findings/issues/210)
- [M-20: Shortfall might be calculated incorrectly if a price value for one collateral isn't fetched correctly](https://github.com/code-423n4/2023-01-reserve-findings/issues/200)
- [M-21: Loss of staking yield for stakers when another user stakes in pause/frozen state](https://github.com/code-423n4/2023-01-reserve-findings/issues/148)
- [M-22: RecollateralizationLib: Dust loss for an asset should be capped at its low value](https://github.com/code-423n4/2023-01-reserve-findings/issues/106)
- [M-23: StRSR: seizeRSR function fails to update rsrRewardsAtLastPayout variable](https://github.com/code-423n4/2023-01-reserve-findings/issues/64)
- [M-24: BasketHandler: Users might not be able to redeem their rToken when protocol is paused due to refreshBasket function](https://github.com/code-423n4/2023-01-reserve-findings/issues/39)
- [M-25: BackingManager: rTokens might not be redeemable when protocol is paused due to missing token allowance](https://github.com/code-423n4/2023-01-reserve-findings/issues/16)

## Overview of changes

The sponsors have made many, many changes, in response to the thoughtful feedback from the Wardens. In most cases changes were straightforward and of limited scope, but in at least two cases there were significant reductions or simplifications of large portions of the code. These areas are expanded upon below in their own sections. The 3rd section will cover everything else:

### 1. Removal of non-atomic RToken issuance (M-13, M-15)

[PR #571: remove non-atomic issuance](https://github.com/reserve-protocol/protocol/pull/571)

This audit, as in previous audits ([ToB](https://github.com/code-423n4/2023-01-reserve/blob/main/audits/Trail%20of%20Bits%20-%20Aug%2011%202022.pdf); [Solidified](https://github.com/code-423n4/2023-01-reserve/blob/main/audits/Solidified%20-%20Oct%2016%202022.pdf)) problems were found with the RToken issuance queue, a fussy cumulative data structure that exists to support constant-time `cancel()` and `vest()` operations for non-atomic issuance. This audit too, another issue was discovered with **M-13**. This prompted us to look for alternatives that achieve a similar purpose to the issuance queue, leading to removal of non-atomic issuance entirely and creation of the issuance throttle. The issuance throttle is at a low-level mechanistically similar to the redemption battery from before, except it is a _net hourly issuance_ measure. This addresses the problem of ingesting large amounts of bad collateral too quickly in a different way and with less frictions for users, both in terms of time and gas fees. 

As wardens will see, large portions of the `RToken` contract code have been removed. This also freed up contract bytecode real estate that allowed us to take libraries internal that were previously external. 

##### Context: Original purpose of issuance queue

The original purpose of the issuance queue was to prevent MEV searchers and other unspeakables from depositing large amounts of collateral right before the basket becomes IFFY and issuance is turned off. The overall IFFY -> DISABLED basket flow can be frontrun, and even though the depositer does not know yet whether a collateral token will default, acquiring a position in the queue acts like a valuable option that pays off if it does and has only opportunity cost otherwise. From the protocol's perspective, this kind of issuance just introduces bad debt. 

The new issunce throttle is purely atomic and serves the same purpose of limiting the loss due to bad debt directly prior to a collateral default. 

### 2. Tightening of the basket range formula (H-02, M-20, M-22)

[PR #585: Narrow bu band](https://github.com/reserve-protocol/protocol/pull/585)

H-02 is the other highly consequential change, from a sheer quantity of SLOC point of view. Indeed, the calculation of the top and bottom of the basket range was highly inefficient and would generally result in larger haircuts than desirable. Below are two datapoints from tests that show the severity of a haircut after default in the absence of RSR overcollateralization:

- **37.5%** loss +  market-even trades
  - Before: **39.7%** haircut
  - After: **37.52%** haircut
- **15%** loss + worst-case below market trades
  - Before: **17.87%** haircut
  - After: **16.38%** haircut

The previous code was more complicated, more costly, and provided worse outcomes. In short this was because it didn't distinguish between capital that needed to be traded vs capital that did not. While the protocol cannot know ahead of time exactly how many BUs it will have after recollateralization, it can use the number of basket units currently held as a bedrock that it knows it will not need to trade, and thus do not differentially contribute to `basket.top` and `basket.bottom`. 

##### Related issues

In addition to H-02 this PR also addressed M-20 and M-22, which are related to the calculation of the dust loss and potential overflow during the shortfall calculation. The calculation of the dust loss is now capped appropriately and the shortfall calculation has been eliminated. 

### 3. Everything else

The mitigations for the remaining issues were more narrow in scope. Most do not require further context or description. But there are 2 smaller clusters of changes worth calling out:

##### Universal Revenue Hiding

[PR #620: Universal revenue hiding](https://github.com/reserve-protocol/protocol/pull/620)

As a warden pointed out in H-01, there are subtleties that can cause the compound v2 cToken rate to decrease, albeit by extremely little. Since we have dealt with Compund V2 for so long, and only just discovered this detail, we reason there are probably more like it. 

To this end we've implemented universal revenue hiding at the collateral plugin level, for all appreciating collateral. The idea is that even a small amount of revenue hiding such as 1-part-in-1-million may end up protecting the collateral plugin from unexpected default while being basically undetectable to humans. 

We mention this change because it can potentially impact other areas of the protocol, such as what prices trades are opened at, or how the basket range is calculated during recollateralization. A warden looking to examine this should focus their attention on `contracts/assets/AppreciatingFiatCollateral.sol`. 

##### Redemption while DISABLED

[PR #575: support redemption while disabled](https://github.com/reserve-protocol/protocol/pull/575)

The final change area to bring attention to is the enabling of RToken redemption while the basket is DISABLED. The motivation for this change is not neatly captured in a single contest issue, though it was something discussed with wardens via DM, and which seems tangentially related to issues like M-03.

Previous behavior: Cannot redeem while DISABLED. `BasketHandler.refreshBasket()` must be called before first redemption can occur, and even then, the redeemer must wait until trading finishes to receive full redemptions. 

Current behavior: Can redeem while DISABLED. Will get full share of collateral until `BasketHandler.refreshBasket()` is called. Can use `revertOnPartialRedemption` redemption param to control behavior along this boundary. 

We mention this change because functionality under different basket conditions is central to the functioning of our protocol. RToken redemption is how capital primarily exits the system, so any change to this area is fundamentally risky. 

A related change is that `BasketHandler._switchBasket()` now skips over IFFY collateral. 

## Mitigations to be reviewed

| URL | Mitigation of | Purpose | 
| ----------- | ------------- | ----------- |
| https://github.com/reserve-protocol/protocol/pull/571 | M-13, M-15 | This PR removes the non-atomic issuance mechanism and adds an issuance throttle. The redemption battery is rebranded to a redemption throttle. |
| https://github.com/reserve-protocol/protocol/pull/585 | H-02, M-20, M-22 | This PR simplifies and improves the basket range formula. The new logic should provide much tighter basket range estimates and result in smaller haircuts. |
| https://github.com/reserve-protocol/protocol/pull/584 | M-01, M-12, M-23, M-25 | This PR bundles mitigations for many small issues together. The full list is in the PR description. Each of these items are small and local in scope. | 
| https://github.com/reserve-protocol/protocol/pull/575 | M-24 | This PR enables redemption while the basket is DISABLED. |
| https://github.com/reserve-protocol/protocol/pull/614 | M-18 | This PR removes the ability to change StRSR token's name and symbol. |
| https://github.com/reserve-protocol/protocol/pull/615 | M-03, M-04 | This PR allows an RToken redeemer to specify when they require full redemptions vs accept partial (prorata) redemptions. |
| https://github.com/reserve-protocol/protocol/pull/617 | M-02 | This PR prevents paying out StRSR rewards until the StRSR supply is at least 1e18. |
| https://github.com/reserve-protocol/protocol/pull/619 | M-05 | This PR prevents melting RToken until the RToken supply is at least 1e18. |
| https://github.com/reserve-protocol/protocol/pull/620 | H-01 | This PR adds universal revenue hiding to all appreciating collateral. |
| https://github.com/reserve-protocol/protocol/pull/622 | M-11 | This PR adds a Furnace.melt()/StRSR.payoutRewards() step when governance changes the rewardRatio. |
| https://github.com/reserve-protocol/protocol/pull/623 | M-16, M-21 | This PR makes the AssetRegistry more resilient to bad collateral during asset unregistration, and disables staking when frozen. |
| https://github.com/reserve-protocol/protocol/pull/628 | M-10 | This PR makes all dangerous uint192 downcasts truncation-safe. |

The sponsors want to emphasize this is _not_ the complete list of changes between the original [df7eca commit](https://github.com/reserve-protocol/protocol/commit/df7ecadc2bae74244ace5e8b39e94bc992903158) and the mitigation review [27a347 commit](https://github.com/reserve-protocol/protocol/commit/27a3472d553b4fa54f896596007765ec91941348). While it is the _vast majority_ of the changes, we urge wardens to check out the diff between the two commits for themselves, as changes may have been made due to addressing gas/QA findings. 
