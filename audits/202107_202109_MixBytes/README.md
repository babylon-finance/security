# Babylon Finance Code Review

## General Information

* Date: From July 2021 to September 2021
* Repository: https://github.com/babylon-finance/protocol
* Commit: d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3

## Visibility

As requested by MixBytes and agreed between both parties after the audit was started, the audit would invest all the time possible to make a deep code review in order to have a more robust protocol vs. generating a public audit report, so we agreed to deep dive into the code although it meant not making the report as public available this time. 

So MixBytes team included the following license which does not allow us to make details public.

**** FOR INTERNAL USE ONLY ****
PLEASE BE AWARE THAT CURRENT DOCUMENT IS A CODE REVIEW FOR THE BABYLON TEAM.
THE DOCUMENT SHOULD NOT BE REFERRED TO AS A PUBLIC AUDIT REPORT UNDER ANY CIRCUMSTANCES AND/OR PRESENTED AS A WARRANTY OF CODE SAFETY.
**** FOR INTERNAL USE ONLY ****

Anyway it was not a bad decision as MixBytes auditors demonstrated having high level security DeFi experts who helped a lot to find even more issues than initially expected.

## Scope of the code review


The scope of the audit includes the following smart contracts at:

https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/BabController.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/GardenValuer.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/PriceOracle.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/gardens/Garden.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/gardens/GardenFactory.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/gardens/GardenNFT.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/strategies/Strategy.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/strategies/StrategyNFT.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/strategies/StrategyFactory.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/Treasury.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/token/RewardsDistributor.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/strategies/operations/Operation.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/strategies/operations/LendOperation.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/strategies/operations/AddLiquidityOperation.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/strategies/operations/BorrowOperation.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/strategies/operations/DepositVaultOperation.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/strategies/operations/BuyOperation.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/BaseIntegration.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/lend/LendIntegration.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/lend/CompoundLendIntegration.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/lend/AaveLendIntegration.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/borrow/BorrowIntegration.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/borrow/CompoundBorrowIntegration.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/borrow/AaveBorrowIntegration.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/passive/PassiveIntegration.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/passive/YearnVaultIntegration.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/passive/HarvestVaultIntegration.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/trade/TradeIntegration.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/trade/UniswapV3TradeIntegration.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/pool/PoolIntegration.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/pool/UniswapPoolIntegration.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/pool/OneInchPoolIntegration.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/pool/BalancerIntegration.sol
https://github.com/babylon-finance/protocol/blob/d0f1f850404a37b7a8630bf62342ca9b1cfb2ed3/contracts/integrations/pool/SushiswapPoolIntegration.sol
