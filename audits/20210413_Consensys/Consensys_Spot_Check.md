# Babylon Finance Spot Check

## General Information

* Date: April 13, 2021
* Repository: https://github.com/babylon-finance/protocol
* Commit: 39626d9a885fe8d0997c5288bc78ecbdfae75ae9

## Overall Impression

Based on the very superficial 1-day engagement for this review, the system's value proposition and architecture seem convincing.

## Documentation

URL: https://app.gitbook.com/@babylon-finance/s/babylon-finance/

The gitbook is well-written, informative, and helpful in understanding the motivation for the system and its design. It takes more of a high-level perspective; there is also some technical information, but more details would probably be helpful for the full audit. Some sections in the book are still empty.

## Individual Remarks

* The function `RollingGarden.reenableEthForInvestments` assumes redemptions are closed if `block.timestamp >= redemptionsOpenUntil`. The function `RollingGarden.canWithdrawEthAmount`, on the other hand, assumes redemptions are open if `block.timestamp <= redemptionsOpenUntil`. This means if the block's timestamp is exactly `redemptionsOpenUntil`, these two functions will come to different conclusions whether redemptions are open or not. (Note that miners have some freedom in choosing timestamps.) Such inconsistencies can lead to major problems because things that should be mutually exclusive can happen together.
One way to avoid this is to use an appropriately named function like `redemptionsOpen` (that returns `true` or `false`) instead of direct comparisons; however, in order to be safe, it has to be used _always_.
If you want to keep using direct comparisons, choose an unambiguous name and stick precisely to its meaning; for example, the name `redemptionsOpenUntil` suggests they're still open at the given value. Or choose a convention throughout the codebase, such as "boundary points always belong to the left" (names should still be given accordingly).
There might be more of such occurrences in the codebase.

* In `RollingGarden.sol`, line 352:
    ```
            reserveAvailableForRedemptionsInWindow.add(_amount);
    ```
    and line 385:
    ```
            totalRequestsAmountInWindow.add(_amount);
    ```
    These should probably be assignments.
    There might be more of those.

* In `RollingGarden.sol`, lines 409-412:
   ```            // Requested a redemption
            if (redemptionRequests[_contributor] > 0) {
                return
                    redemptionRequests[_contributor].div(totalRequestsAmountInWindow).mul(
                        reserveAvailableForRedemptionsInWindow
                    ) >= _amount;
            }
   ```
   Currently, due to the bug mentioned in the previous item, `totalRequestsAmountInWindow` will always be 0. Consequently, the `div` will revert whenever we get there, so the case of a requested redemption seems to be untested.
Even without the bug above, by dividing first and multiplying later we will still end up with 0, unless there's only one redemption request.

* A general concern are permanent reverts due to SafeMath. While SafeMath protects against unnoticed over- and underflows, just reverting is sometimes not good enough.
Imagine in the formula above, we multiply first and divide later. If `redemptionRequests[_contributor].mul(reserveAvailableForRedemptionsInWindow)` reverts, it will always do so whenever we get there (for the same contributor) and therefore could permanently block an important operation and thereby freeze funds or even halt an entire contract.
This is just an example; the limited supply of Ether and other factors might prevent that from happening in a particular scenario. But as soon as you include other assets, assumptions about their total supply that were valid for Ether may no longer hold.
    
## Tests

All 168 tests pass.

Coverage report:
```
--------------------------------------------|----------|----------|----------|----------|----------------|
File                                        |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
--------------------------------------------|----------|----------|----------|----------|----------------|
 contracts/                                 |    84.85 |    51.28 |    85.71 |    84.34 |                |
  BabController.sol                         |    84.76 |       42 |    86.36 |    83.96 |... 410,428,502 |
  GardenValuer.sol                          |     87.5 |       50 |      100 |     87.5 |        101,102 |
  PriceOracle.sol                           |    81.58 |       60 |       75 |    81.58 |... 139,141,226 |
  Treasury.sol                              |      100 |      100 |      100 |      100 |                |
 contracts/gardens/                         |    81.82 |    44.44 |    80.39 |    81.99 |                |
  BaseGarden.sol                            |       76 |       25 |    78.57 |    77.11 |... 425,451,497 |
  GardenFactory.sol                         |      100 |      100 |      100 |      100 |                |
  RollingGarden.sol                         |    83.91 |    46.88 |    80.95 |    83.91 |... 664,665,666 |
 contracts/governance/                      |    49.28 |    29.41 |    61.11 |    48.57 |                |
  VoteToken.sol                             |    49.28 |    29.41 |    61.11 |    48.57 |... 339,340,343 |
 contracts/integrations/                    |      100 |       50 |      100 |      100 |                |
  BaseIntegration.sol                       |      100 |       50 |      100 |      100 |                |
 contracts/integrations/lend/               |    76.74 |    23.53 |    73.68 |    75.86 |                |
  AaveLendIntegration.sol                   |      100 |      100 |      100 |      100 |                |
  CompoundLendIntegration.sol               |    84.62 |       25 |       80 |    81.48 |... ,85,115,116 |
  LendIntegration.sol                       |    67.35 |    23.33 |       60 |    67.35 |... 347,353,354 |
 contracts/integrations/passive/            |    76.27 |       25 |    74.07 |    76.27 |                |
  PassiveIntegration.sol                    |    70.83 |       25 |    63.16 |    70.83 |... 368,374,375 |
  YearnVaultIntegration.sol                 |      100 |      100 |      100 |      100 |                |
 contracts/integrations/pool/               |    88.19 |    40.38 |    86.05 |    88.19 |                |
  BalancerIntegration.sol                   |      100 |       50 |      100 |      100 |                |
  OneInchPoolIntegration.sol                |    96.67 |       50 |      100 |    96.67 |            159 |
  PoolIntegration.sol                       |    70.83 |    32.14 |       60 |    70.83 |... 314,320,321 |
  SushiswapPoolIntegration.sol              |      100 |      100 |      100 |      100 |                |
  UniswapPoolIntegration.sol                |      100 |       50 |      100 |      100 |                |
 contracts/integrations/trade/              |     76.6 |    28.57 |    56.25 |       75 |                |
  KyberTradeIntegration.sol                 |    16.67 |      100 |       25 |    16.67 |... 114,126,136 |
  OneInchTradeIntegration.sol               |    71.43 |        0 |       60 |     62.5 |       41,42,70 |
  TradeIntegration.sol                      |    88.24 |    33.33 |    71.43 |    88.24 |236,237,246,247 |
 contracts/interfaces/                      |      100 |      100 |      100 |      100 |                |
  IBabController.sol                        |      100 |      100 |      100 |      100 |                |
  IBorrowIntegration.sol                    |      100 |      100 |      100 |      100 |                |
  ICoverIntegration.sol                     |      100 |      100 |      100 |      100 |                |
  IFarmIntegration.sol                      |      100 |      100 |      100 |      100 |                |
  IGarden.sol                               |      100 |      100 |      100 |      100 |                |
  IGardenFactory.sol                        |      100 |      100 |      100 |      100 |                |
  IGardenValuer.sol                         |      100 |      100 |      100 |      100 |                |
  IIntegration.sol                          |      100 |      100 |      100 |      100 |                |
  ILendIntegration.sol                      |      100 |      100 |      100 |      100 |                |
  IOracleAdapter.sol                        |      100 |      100 |      100 |      100 |                |
  IPassiveIntegration.sol                   |      100 |      100 |      100 |      100 |                |
  IPoolIntegration.sol                      |      100 |      100 |      100 |      100 |                |
  IPriceOracle.sol                          |      100 |      100 |      100 |      100 |                |
  IRewardsDistributor.sol                   |      100 |      100 |      100 |      100 |                |
  IRollingGarden.sol                        |      100 |      100 |      100 |      100 |                |
  IStrategy.sol                             |      100 |      100 |      100 |      100 |                |
  IStrategyFactory.sol                      |      100 |      100 |      100 |      100 |                |
  ITimelock.sol                             |      100 |      100 |      100 |      100 |                |
  ITradeIntegration.sol                     |      100 |      100 |      100 |      100 |                |
  IUniswapAnchoredView.sol                  |      100 |      100 |      100 |      100 |                |
  IVoteToken.sol                            |      100 |      100 |      100 |      100 |                |
 contracts/interfaces/external/1inch/       |      100 |      100 |      100 |      100 |                |
  IMooniswap.sol                            |      100 |      100 |      100 |      100 |                |
  IMooniswapFactory.sol                     |      100 |      100 |      100 |      100 |                |
  IMooniswapFactoryGovernance.sol           |      100 |      100 |      100 |      100 |                |
  IOneInchExchange.sol                      |      100 |      100 |      100 |      100 |                |
 contracts/interfaces/external/aave/        |      100 |      100 |      100 |      100 |                |
  AaveToken.sol                             |      100 |      100 |      100 |      100 |                |
  DataTypes.sol                             |      100 |      100 |      100 |      100 |                |
  ILendingPool.sol                          |      100 |      100 |      100 |      100 |                |
  ILendingPoolAddressesProvider.sol         |      100 |      100 |      100 |      100 |                |
  ILendingPoolAddressesProviderRegistry.sol |      100 |      100 |      100 |      100 |                |
  IProtocolDataProvider.sol                 |      100 |      100 |      100 |      100 |                |
  IStableDebtToken.sol                      |      100 |      100 |      100 |      100 |                |
  Oracle.sol                                |      100 |      100 |      100 |      100 |                |
 contracts/interfaces/external/balancer/    |      100 |      100 |      100 |      100 |                |
  IBFactory.sol                             |      100 |      100 |      100 |      100 |                |
  IBPool.sol                                |      100 |      100 |      100 |      100 |                |
 contracts/interfaces/external/compound/    |      100 |      100 |      100 |      100 |                |
  ICEther.sol                               |      100 |      100 |      100 |      100 |                |
  ICToken.sol                               |      100 |      100 |      100 |      100 |                |
  ICompoundPriceOracle.sol                  |      100 |      100 |      100 |      100 |                |
  IComptroller.sol                          |      100 |      100 |      100 |      100 |                |
 contracts/interfaces/external/kyber/       |      100 |      100 |      100 |      100 |                |
  IKyberNetworkProxy.sol                    |      100 |      100 |      100 |      100 |                |
 contracts/interfaces/external/uniswap/     |      100 |      100 |      100 |      100 |                |
  IUniswapV2PairB.sol                       |      100 |      100 |      100 |      100 |                |
  IUniswapV2Router.sol                      |      100 |      100 |      100 |      100 |                |
 contracts/interfaces/external/weth/        |      100 |      100 |      100 |      100 |                |
  IWETH.sol                                 |      100 |      100 |      100 |      100 |                |
 contracts/interfaces/external/yearn/       |      100 |      100 |      100 |      100 |                |
  IController.sol                           |      100 |      100 |      100 |      100 |                |
  IConverter.sol                            |      100 |      100 |      100 |      100 |                |
  IDelegatedVault.sol                       |      100 |      100 |      100 |      100 |                |
  IGovernance.sol                           |      100 |      100 |      100 |      100 |                |
  IOneSplitAudit.sol                        |      100 |      100 |      100 |      100 |                |
  IProxy.sol                                |      100 |      100 |      100 |      100 |                |
  IStrategy.sol                             |      100 |      100 |      100 |      100 |                |
  IToken.sol                                |      100 |      100 |      100 |      100 |                |
  IVault.sol                                |      100 |      100 |      100 |      100 |                |
  IVoterProxy.sol                           |      100 |      100 |      100 |      100 |                |
  IWrappedVault.sol                         |      100 |      100 |      100 |      100 |                |
  YRegistry.sol                             |      100 |      100 |      100 |      100 |                |
 contracts/lib/                             |      100 |      100 |      100 |      100 |                |
  BabylonErrors.sol                         |      100 |      100 |      100 |      100 |                |
 contracts/oracle_adapter/                  |      100 |       75 |      100 |      100 |                |
  UniswapTWAP.sol                           |      100 |       75 |      100 |      100 |                |
 contracts/strategies/                      |    91.79 |    51.89 |    81.63 |    91.12 |                |
  LendStrategy.sol                          |      100 |       50 |      100 |      100 |                |
  LendStrategyFactory.sol                   |      100 |      100 |      100 |      100 |                |
  LiquidityPoolStrategy.sol                 |    96.88 |    78.57 |      100 |    96.88 |             75 |
  LiquidityPoolStrategyFactory.sol          |      100 |      100 |      100 |      100 |                |
  LongStrategy.sol                          |      100 |       50 |      100 |      100 |                |
  LongStrategyFactory.sol                   |      100 |      100 |      100 |      100 |                |
  Strategy.sol                              |    86.99 |     47.3 |    68.97 |    86.15 |... 499,500,501 |
  YieldFarmingStrategy.sol                  |      100 |       50 |      100 |      100 |                |
  YieldFarmingStrategyFactory.sol           |      100 |      100 |      100 |      100 |                |
 contracts/token/                           |    89.93 |    80.14 |    83.67 |    90.94 |                |
  BABLToken.sol                             |      100 |      100 |      100 |      100 |                |
  RewardsDistributor.sol                    |    81.15 |       70 |    66.67 |    82.64 |... 319,324,479 |
  TimeLockRegistry.sol                      |      100 |     87.5 |      100 |      100 |                |
  TimeLockedToken.sol                       |    94.81 |    77.27 |    93.33 |    95.06 |252,287,306,313 |
--------------------------------------------|----------|----------|----------|----------|----------------|
All files                                   |    84.17 |    53.28 |    79.89 |    84.13 |                |
--------------------------------------------|----------|----------|----------|----------|----------------|
```

## MythX Analysis

```
Report for node_modules/@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol
╒════════╤════════════════════════════════════════╤════════════╤═══════════════════════════════════════╕
│   Line │ SWC Title                              │ Severity   │ Short Description                     │
╞════════╪════════════════════════════════════════╪════════════╪═══════════════════════════════════════╡
│     60 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│     69 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│     74 │ (SWC-131) Presence of unused variables │ Low        │ Unused state variable "__gap".        │
╘════════╧════════════════════════════════════════╧════════════╧═══════════════════════════════════════╛

Report for node_modules/@openzeppelin/contracts-upgradeable/utils/ContextUpgradeable.sol
╒════════╤════════════════════════════════════════╤════════════╤════════════════════════════════╕
│   Line │ SWC Title                              │ Severity   │ Short Description              │
╞════════╪════════════════════════════════════════╪════════════╪════════════════════════════════╡
│     31 │ (SWC-131) Presence of unused variables │ Low        │ Unused state variable "__gap". │
╘════════╧════════════════════════════════════════╧════════════╧════════════════════════════════╛

Report for node_modules/@openzeppelin/contracts/math/SafeMath.sol
╒════════╤════════════════════════════════╤════════════╤════════════════════════════════════════════════════════════════════════════════════╕
│   Line │ SWC Title                      │ Severity   │ Short Description                                                                  │
╞════════╪════════════════════════════════╪════════════╪════════════════════════════════════════════════════════════════════════════════════╡
│     87 │ (SWC-116) Timestamp Dependence │ Low        │ A control flow decision is made based on The block.timestamp environment variable. │
╘════════╧════════════════════════════════╧════════════╧════════════════════════════════════════════════════════════════════════════════════╛

Report for node_modules/@openzeppelin/contracts/token/ERC20/ERC20.sol
╒════════╤════════════════════════════════════════╤════════════╤═══════════════════════════════════════╕
│   Line │ SWC Title                              │ Severity   │ Short Description                     │
╞════════╪════════════════════════════════════════╪════════════╪═══════════════════════════════════════╡
│     64 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│     72 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│     89 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│     96 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    103 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    115 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    123 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    134 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    152 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    170 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    189 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    305 │ (SWC-131) Presence of unused variables │ Low        │ Unused function parameter "amount".   │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    305 │ (SWC-131) Presence of unused variables │ Low        │ Unused function parameter "to".       │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    305 │ (SWC-131) Presence of unused variables │ Low        │ Unused function parameter "from".     │
╘════════╧════════════════════════════════════════╧════════════╧═══════════════════════════════════════╛

Report for contracts/GardenValuer.sol
╒════════╤════════════════════════════════════════╤════════════╤═════════════════════════════════════╕
│   Line │ SWC Title                              │ Severity   │ Short Description                   │
╞════════╪════════════════════════════════════════╪════════════╪═════════════════════════════════════╡
│     64 │ (SWC-100) Function Default Visibility  │ Low        │ Function visibility is not set.     │
├────────┼────────────────────────────────────────┼────────────┼─────────────────────────────────────┤
│     92 │ (SWC-128) DoS With Block Gas Limit     │ Low        │ Loop over unbounded data structure. │
├────────┼────────────────────────────────────────┼────────────┼─────────────────────────────────────┤
│     95 │ (SWC-131) Presence of unused variables │ Low        │ Unused local variable "strategy".   │
╘════════╧════════════════════════════════════════╧════════════╧═════════════════════════════════════╛

Report for contracts/lib/PreciseUnitMath.sol
╒════════╤════════════════════════════════════╤════════════╤═════════════════════════════════════╕
│   Line │ SWC Title                          │ Severity   │ Short Description                   │
╞════════╪════════════════════════════════════╪════════════╪═════════════════════════════════════╡
│    171 │ (SWC-128) DoS With Block Gas Limit │ Low        │ Loop over unbounded data structure. │
╘════════╧════════════════════════════════════╧════════════╧═════════════════════════════════════╛

Report for node_modules/@openzeppelin/contracts/access/Ownable.sol
╒════════╤═══════════════════╤════════════╤═══════════════════════════════════════╕
│   Line │ SWC Title         │ Severity   │ Short Description                     │
╞════════╪═══════════════════╪════════════╪═══════════════════════════════════════╡
│     54 │ (SWC-000) Unknown │ Medium     │ Function could be marked as external. │
├────────┼───────────────────┼────────────┼───────────────────────────────────────┤
│     63 │ (SWC-000) Unknown │ Medium     │ Function could be marked as external. │
╘════════╧═══════════════════╧════════════╧═══════════════════════════════════════╛

Report for node_modules/@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol
╒════════╤════════════════════════════════════════╤════════════╤═══════════════════════════════════════╕
│   Line │ SWC Title                              │ Severity   │ Short Description                     │
╞════════╪════════════════════════════════════════╪════════════╪═══════════════════════════════════════╡
│     70 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│     78 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│     95 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    102 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    109 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    121 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    129 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    140 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    158 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    176 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    195 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    311 │ (SWC-131) Presence of unused variables │ Low        │ Unused function parameter "to".       │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    311 │ (SWC-131) Presence of unused variables │ Low        │ Unused function parameter "from".     │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    311 │ (SWC-131) Presence of unused variables │ Low        │ Unused function parameter "amount".   │
╘════════╧════════════════════════════════════════╧════════════╧═══════════════════════════════════════╛

Report for node_modules/@openzeppelin/contracts/utils/ReentrancyGuard.sol
╒════════╤════════════════════════════════════════╤════════════╤══════════════════════════════════╕
│   Line │ SWC Title                              │ Severity   │ Short Description                │
╞════════╪════════════════════════════════════════╪════════════╪══════════════════════════════════╡
│     36 │ (SWC-131) Presence of unused variables │ Low        │ Unused state variable "_status". │
╘════════╧════════════════════════════════════════╧════════════╧══════════════════════════════════╛

Report for contracts/gardens/BaseGarden.sol
╒════════╤════════════════════════════════════╤════════════╤══════════════════════════════════════════════╕
│   Line │ SWC Title                          │ Severity   │ Short Description                            │
╞════════╪════════════════════════════════════╪════════════╪══════════════════════════════════════════════╡
│    439 │ (SWC-000) Unknown                  │ Medium     │ Function could be marked as external.        │
├────────┼────────────────────────────────────┼────────────┼──────────────────────────────────────────────┤
│    440 │ (SWC-128) DoS With Block Gas Limit │ Low        │ Implicit loop over unbounded data structure. │
├────────┼────────────────────────────────────┼────────────┼──────────────────────────────────────────────┤
│    449 │ (SWC-000) Unknown                  │ Medium     │ Function could be marked as external.        │
├────────┼────────────────────────────────────┼────────────┼──────────────────────────────────────────────┤
│    450 │ (SWC-128) DoS With Block Gas Limit │ Low        │ Implicit loop over unbounded data structure. │
├────────┼────────────────────────────────────┼────────────┼──────────────────────────────────────────────┤
│    457 │ (SWC-000) Unknown                  │ Medium     │ Function could be marked as external.        │
╘════════╧════════════════════════════════════╧════════════╧══════════════════════════════════════════════╛

Report for contracts/governance/GovernorAlpha.sol
╒════════╤════════════════════════════════════════════════════════════╤════════════╤═══════════════════════════════════════════════════════════╕
│   Line │ SWC Title                                                  │ Severity   │ Short Description                                         │
╞════════╪════════════════════════════════════════════════════════════╪════════════╪═══════════════════════════════════════════════════════════╡
│    189 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    197 │ (SWC-120) Weak Sources of Randomness from Chain Attributes │ Low        │ Potential use of "block.number" as source of randonmness. │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    222 │ (SWC-120) Weak Sources of Randomness from Chain Attributes │ Low        │ Potential use of "block.number" as source of randonmness. │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    264 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    308 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    334 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    348 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    370 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    378 │ (SWC-120) Weak Sources of Randomness from Chain Attributes │ Low        │ Potential use of "block.number" as source of randonmness. │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    401 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    412 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    423 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    434 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    453 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    475 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    490 │ (SWC-120) Weak Sources of Randomness from Chain Attributes │ Low        │ Potential use of "block.number" as source of randonmness. │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    492 │ (SWC-120) Weak Sources of Randomness from Chain Attributes │ Low        │ Potential use of "block.number" as source of randonmness. │
╘════════╧════════════════════════════════════════════════════════════╧════════════╧═══════════════════════════════════════════════════════════╛

Report for contracts/governance/TimeLock.sol
╒════════╤════════════════════════════════════╤════════════╤═════════════════════════════════════════════════════════╕
│   Line │ SWC Title                          │ Severity   │ Short Description                                       │
╞════════╪════════════════════════════════════╪════════════╪═════════════════════════════════════════════════════════╡
│     88 │ (SWC-000) Unknown                  │ Medium     │ Function could be marked as external.                   │
├────────┼────────────────────────────────────┼────────────┼─────────────────────────────────────────────────────────┤
│     97 │ (SWC-000) Unknown                  │ Medium     │ Function could be marked as external.                   │
├────────┼────────────────────────────────────┼────────────┼─────────────────────────────────────────────────────────┤
│    105 │ (SWC-000) Unknown                  │ Medium     │ Function could be marked as external.                   │
├────────┼────────────────────────────────────┼────────────┼─────────────────────────────────────────────────────────┤
│    112 │ (SWC-000) Unknown                  │ Medium     │ Function could be marked as external.                   │
├────────┼────────────────────────────────────┼────────────┼─────────────────────────────────────────────────────────┤
│    132 │ (SWC-000) Unknown                  │ Medium     │ Function could be marked as external.                   │
├────────┼────────────────────────────────────┼────────────┼─────────────────────────────────────────────────────────┤
│    147 │ (SWC-000) Unknown                  │ Medium     │ Function could be marked as external.                   │
├────────┼────────────────────────────────────┼────────────┼─────────────────────────────────────────────────────────┤
│    168 │ (SWC-128) DoS With Block Gas Limit │ Low        │ Potentially unbounded data structure passed to builtin. │
╘════════╧════════════════════════════════════╧════════════╧═════════════════════════════════════════════════════════╛

Report for contracts/governance/VoteToken.sol
╒════════╤════════════════════════════════════════════════════════════╤════════════╤═══════════════════════════════════════════════════════════╕
│   Line │ SWC Title                                                  │ Severity   │ Short Description                                         │
╞════════╪════════════════════════════════════════════════════════════╪════════════╪═══════════════════════════════════════════════════════════╡
│     95 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    111 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    120 │ (SWC-128) DoS With Block Gas Limit                         │ Low        │ Potentially unbounded data structure passed to builtin.   │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    138 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    150 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    151 │ (SWC-120) Weak Sources of Randomness from Chain Attributes │ Low        │ Potential use of "block.number" as source of randonmness. │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    170 │ (SWC-128) DoS With Block Gas Limit                         │ Low        │ Loop over unbounded data structure.                       │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    184 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    188 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    192 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    197 │ (SWC-000) Unknown                                          │ Medium     │ Function could be marked as external.                     │
├────────┼────────────────────────────────────────────────────────────┼────────────┼───────────────────────────────────────────────────────────┤
│    272 │ (SWC-120) Weak Sources of Randomness from Chain Attributes │ Low        │ Potential use of "block.number" as source of randonmness. │
╘════════╧════════════════════════════════════════════════════════════╧════════════╧═══════════════════════════════════════════════════════════╛

Report for contracts/integrations/lend/AaveLendIntegration.sol
╒════════╤═══════════════════════════════════════╤════════════╤═════════════════════════════════╕
│   Line │ SWC Title                             │ Severity   │ Short Description               │
╞════════╪═══════════════════════════════════════╪════════════╪═════════════════════════════════╡
│     67 │ (SWC-100) Function Default Visibility │ Low        │ Function visibility is not set. │
╘════════╧═══════════════════════════════════════╧════════════╧═════════════════════════════════╛

Report for contracts/interfaces/external/aave/DataTypes.sol
╒════════╤═══════════════════════════╤════════════╤═══════════════════════════╕
│   Line │ SWC Title                 │ Severity   │ Short Description         │
╞════════╪═══════════════════════════╪════════════╪═══════════════════════════╡
│      2 │ (SWC-103) Floating Pragma │ Low        │ A floating pragma is set. │
╘════════╧═══════════════════════════╧════════════╧═══════════════════════════╛

Report for contracts/integrations/lend/CompoundLendIntegration.sol
╒════════╤═══════════════════════════════════════╤════════════╤═════════════════════════════════╕
│   Line │ SWC Title                             │ Severity   │ Short Description               │
╞════════╪═══════════════════════════════════════╪════════════╪═════════════════════════════════╡
│     71 │ (SWC-100) Function Default Visibility │ Low        │ Function visibility is not set. │
╘════════╧═══════════════════════════════════════╧════════════╧═════════════════════════════════╛

Report for contracts/integrations/passive/YearnVaultIntegration.sol
╒════════╤═══════════════════════════════════════╤════════════╤═════════════════════════════════╕
│   Line │ SWC Title                             │ Severity   │ Short Description               │
╞════════╪═══════════════════════════════════════╪════════════╪═════════════════════════════════╡
│     53 │ (SWC-100) Function Default Visibility │ Low        │ Function visibility is not set. │
╘════════╧═══════════════════════════════════════╧════════════╧═════════════════════════════════╛

Report for contracts/integrations/pool/BalancerIntegration.sol
╒════════╤═══════════════════════════════════════╤════════════╤═════════════════════════════════╕
│   Line │ SWC Title                             │ Severity   │ Short Description               │
╞════════╪═══════════════════════════════════════╪════════════╪═════════════════════════════════╡
│     53 │ (SWC-100) Function Default Visibility │ Low        │ Function visibility is not set. │
╘════════╧═══════════════════════════════════════╧════════════╧═════════════════════════════════╛

Report for contracts/integrations/pool/PoolIntegration.sol
╒════════╤═══════════════════╤════════════╤═══════════════════════════════════════════════════════╕
│   Line │ SWC Title         │ Severity   │ Short Description                                     │
╞════════╪═══════════════════╪════════════╪═══════════════════════════════════════════════════════╡
│    180 │ (SWC-000) Unknown │ Medium     │ Incorrect function "_createPoolInfo" state mutability │
╘════════╧═══════════════════╧════════════╧═══════════════════════════════════════════════════════╛

Report for contracts/integrations/pool/OneInchPoolIntegration.sol
╒════════╤═══════════════════════════════════════╤════════════╤═════════════════════════════════╕
│   Line │ SWC Title                             │ Severity   │ Short Description               │
╞════════╪═══════════════════════════════════════╪════════════╪═════════════════════════════════╡
│     56 │ (SWC-100) Function Default Visibility │ Low        │ Function visibility is not set. │
╘════════╧═══════════════════════════════════════╧════════════╧═════════════════════════════════╛

Report for contracts/integrations/pool/SushiswapPoolIntegration.sol
╒════════╤═══════════════════════════════════════╤════════════╤═════════════════════════════════╕
│   Line │ SWC Title                             │ Severity   │ Short Description               │
╞════════╪═══════════════════════════════════════╪════════════╪═════════════════════════════════╡
│     40 │ (SWC-100) Function Default Visibility │ Low        │ Function visibility is not set. │
╘════════╧═══════════════════════════════════════╧════════════╧═════════════════════════════════╛

Report for contracts/integrations/pool/UniswapPoolIntegration.sol
╒════════╤═════════════════════════════════════════════╤════════════╤═══════════════════════════════════════╕
│   Line │ SWC Title                                   │ Severity   │ Short Description                     │
╞════════╪═════════════════════════════════════════════╪════════════╪═══════════════════════════════════════╡
│     46 │ (SWC-108) State Variable Default Visibility │ Low        │ State variable visibility is not set. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│     57 │ (SWC-100) Function Default Visibility       │ Low        │ Function visibility is not set.       │
╘════════╧═════════════════════════════════════════════╧════════════╧═══════════════════════════════════════╛

Report for contracts/integrations/trade/KyberTradeIntegration.sol
╒════════╤═══════════════════════════════════════╤════════════╤═════════════════════════════════╕
│   Line │ SWC Title                             │ Severity   │ Short Description               │
╞════════╪═══════════════════════════════════════╪════════════╪═════════════════════════════════╡
│     52 │ (SWC-100) Function Default Visibility │ Low        │ Function visibility is not set. │
╘════════╧═══════════════════════════════════════╧════════════╧═════════════════════════════════╛

Report for contracts/integrations/trade/TradeIntegration.sol
╒════════╤═══════════════════╤════════════╤════════════════════════════════════════════════════════╕
│   Line │ SWC Title         │ Severity   │ Short Description                                      │
╞════════╪═══════════════════╪════════════╪════════════════════════════════════════════════════════╡
│    138 │ (SWC-000) Unknown │ Medium     │ Incorrect function "_createTradeInfo" state mutability │
╘════════╧═══════════════════╧════════════╧════════════════════════════════════════════════════════╛

Report for contracts/integrations/trade/OneInchTradeIntegration.sol
╒════════╤═══════════════════════════════════════╤════════════╤═════════════════════════════════╕
│   Line │ SWC Title                             │ Severity   │ Short Description               │
╞════════╪═══════════════════════════════════════╪════════════╪═════════════════════════════════╡
│     59 │ (SWC-100) Function Default Visibility │ Low        │ Function visibility is not set. │
╘════════╧═══════════════════════════════════════╧════════════╧═════════════════════════════════╛

Report for ILendingPool.sol
╒════════╤═══════════════════════════╤════════════╤═══════════════════════════╕
│   Line │ SWC Title                 │ Severity   │ Short Description         │
╞════════╪═══════════════════════════╪════════════╪═══════════════════════════╡
│      2 │ (SWC-103) Floating Pragma │ Low        │ A floating pragma is set. │
╘════════╧═══════════════════════════╧════════════╧═══════════════════════════╛

Report for contracts/lib/Math.sol
╒════════╤════════════════════════════════════╤════════════╤═════════════════════════════════════╕
│   Line │ SWC Title                          │ Severity   │ Short Description                   │
╞════════╪════════════════════════════════════╪════════════╪═════════════════════════════════════╡
│     41 │ (SWC-128) DoS With Block Gas Limit │ Low        │ Loop over unbounded data structure. │
╘════════╧════════════════════════════════════╧════════════╧═════════════════════════════════════╛

Report for contracts/mocks/BabControllerV2Mock.sol
╒════════╤════════════════════════════════════════╤════════════╤═══════════════════════════════════════╕
│   Line │ SWC Title                              │ Severity   │ Short Description                     │
╞════════╪════════════════════════════════════════╪════════════╪═══════════════════════════════════════╡
│     60 │ (SWC-131) Presence of unused variables │ Low        │ Unused state variable "integrations". │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    100 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    112 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
╘════════╧════════════════════════════════════════╧════════════╧═══════════════════════════════════════╛

Report for contracts/oracle_adapter/UniswapTWAP.sol
╒════════╤═══════════════════════════════════════╤════════════╤═════════════════════════════════╕
│   Line │ SWC Title                             │ Severity   │ Short Description               │
╞════════╪═══════════════════════════════════════╪════════════╪═════════════════════════════════╡
│     87 │ (SWC-100) Function Default Visibility │ Low        │ Function visibility is not set. │
╘════════╧═══════════════════════════════════════╧════════════╧═════════════════════════════════╛

Report for contracts/strategies/LendStrategy.sol
╒════════╤═════════════════════════════════╤════════════╤═══════════════════════════════════════╕
│   Line │ SWC Title                       │ Severity   │ Short Description                     │
╞════════╪═════════════════════════════════╪════════════╪═══════════════════════════════════════╡
│     37 │ (SWC-123) Requirement Violation │ Low        │ Requirement violation.                │
├────────┼─────────────────────────────────┼────────────┼───────────────────────────────────────┤
│     48 │ (SWC-000) Unknown               │ Medium     │ Function could be marked as external. │
╘════════╧═════════════════════════════════╧════════════╧═══════════════════════════════════════╛

Report for contracts/strategies/Strategy.sol
╒════════╤════════════════════════════════════╤════════════╤══════════════════════════════════════════════╕
│   Line │ SWC Title                          │ Severity   │ Short Description                            │
╞════════╪════════════════════════════════════╪════════════╪══════════════════════════════════════════════╡
│    191 │ (SWC-000) Unknown                  │ Medium     │ Function could be marked as external.        │
├────────┼────────────────────────────────────┼────────────┼──────────────────────────────────────────────┤
│    205 │ (SWC-123) Requirement Violation    │ Low        │ Requirement violation.                       │
├────────┼────────────────────────────────────┼────────────┼──────────────────────────────────────────────┤
│    254 │ (SWC-128) DoS With Block Gas Limit │ Medium     │ Implicit loop over unbounded data structure. │
├────────┼────────────────────────────────────┼────────────┼──────────────────────────────────────────────┤
│    271 │ (SWC-000) Unknown                  │ Medium     │ Function could be marked as external.        │
╘════════╧════════════════════════════════════╧════════════╧══════════════════════════════════════════════╛

Report for contracts/strategies/LendStrategyFactory.sol
╒════════╤═════════════════════════════════════════════╤════════════╤═══════════════════════════════════════╕
│   Line │ SWC Title                                   │ Severity   │ Short Description                     │
╞════════╪═════════════════════════════════════════════╪════════════╪═══════════════════════════════════════╡
│     34 │ (SWC-108) State Variable Default Visibility │ Low        │ State variable visibility is not set. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│     36 │ (SWC-100) Function Default Visibility       │ Low        │ Function visibility is not set.       │
╘════════╧═════════════════════════════════════════════╧════════════╧═══════════════════════════════════════╛

Report for contracts/strategies/LiquidityPoolStrategy.sol
╒════════╤════════════════════════════════════════╤════════════╤══════════════════════════════════════════════╕
│   Line │ SWC Title                              │ Severity   │ Short Description                            │
╞════════╪════════════════════════════════════════╪════════════╪══════════════════════════════════════════════╡
│     36 │ (SWC-123) Requirement Violation        │ Low        │ Requirement violation.                       │
├────────┼────────────────────────────────────────┼────────────┼──────────────────────────────────────────────┤
│     48 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external.        │
├────────┼────────────────────────────────────────┼────────────┼──────────────────────────────────────────────┤
│     54 │ (SWC-128) DoS With Block Gas Limit     │ Medium     │ Implicit loop over unbounded data structure. │
├────────┼────────────────────────────────────────┼────────────┼──────────────────────────────────────────────┤
│     66 │ (SWC-131) Presence of unused variables │ Low        │ Unused local variable "ethValue".            │
├────────┼────────────────────────────────────────┼────────────┼──────────────────────────────────────────────┤
│     67 │ (SWC-128) DoS With Block Gas Limit     │ Medium     │ Loop over unbounded data structure.          │
├────────┼────────────────────────────────────────┼────────────┼──────────────────────────────────────────────┤
│     88 │ (SWC-128) DoS With Block Gas Limit     │ Medium     │ Implicit loop over unbounded data structure. │
├────────┼────────────────────────────────────────┼────────────┼──────────────────────────────────────────────┤
│    102 │ (SWC-128) DoS With Block Gas Limit     │ Medium     │ Implicit loop over unbounded data structure. │
├────────┼────────────────────────────────────────┼────────────┼──────────────────────────────────────────────┤
│    107 │ (SWC-128) DoS With Block Gas Limit     │ Medium     │ Loop over unbounded data structure.          │
╘════════╧════════════════════════════════════════╧════════════╧══════════════════════════════════════════════╛

Report for contracts/strategies/LongStrategy.sol
╒════════╤═══════════════════╤════════════╤═══════════════════════════════════════╕
│   Line │ SWC Title         │ Severity   │ Short Description                     │
╞════════╪═══════════════════╪════════════╪═══════════════════════════════════════╡
│     41 │ (SWC-000) Unknown │ Medium     │ Function could be marked as external. │
╘════════╧═══════════════════╧════════════╧═══════════════════════════════════════╛

Report for contracts/strategies/YieldFarmingStrategy.sol
╒════════╤═══════════════════╤════════════╤═══════════════════════════════════════╕
│   Line │ SWC Title         │ Severity   │ Short Description                     │
╞════════╪═══════════════════╪════════════╪═══════════════════════════════════════╡
│     47 │ (SWC-000) Unknown │ Medium     │ Function could be marked as external. │
╘════════╧═══════════════════╧════════════╧═══════════════════════════════════════╛

Report for contracts/token/BABLToken.sol
╒════════╤═══════════════════════════════════════╤════════════╤═══════════════════════════════════════╕
│   Line │ SWC Title                             │ Severity   │ Short Description                     │
╞════════╪═══════════════════════════════════════╪════════════╪═══════════════════════════════════════╡
│     89 │ (SWC-100) Function Default Visibility │ Low        │ Function visibility is not set.       │
├────────┼───────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    210 │ (SWC-000) Unknown                     │ Medium     │ Function could be marked as external. │
╘════════╧═══════════════════════════════════════╧════════════╧═══════════════════════════════════════╛

Report for contracts/token/TimeLockRegistry.sol
╒════════╤════════════════════════════════════════╤════════════╤═══════════════════════════════════════╕
│   Line │ SWC Title                              │ Severity   │ Short Description                     │
╞════════╪════════════════════════════════════════╪════════════╪═══════════════════════════════════════╡
│     85 │ (SWC-131) Presence of unused variables │ Low        │ Unused state variable "vestingCliff". │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    101 │ (SWC-100) Function Default Visibility  │ Low        │ Function visibility is not set.       │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    199 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    215 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    261 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
├────────┼────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    279 │ (SWC-000) Unknown                      │ Medium     │ Function could be marked as external. │
╘════════╧════════════════════════════════════════╧════════════╧═══════════════════════════════════════╛

Report for contracts/token/TimeLockedToken.sol
╒════════╤═══════════════════╤════════════╤═══════════════════════════════════════╕
│   Line │ SWC Title         │ Severity   │ Short Description                     │
╞════════╪═══════════════════╪════════════╪═══════════════════════════════════════╡
│    202 │ (SWC-000) Unknown │ Medium     │ Function could be marked as external. │
├────────┼───────────────────┼────────────┼───────────────────────────────────────┤
│    286 │ (SWC-000) Unknown │ Medium     │ Function could be marked as external. │
├────────┼───────────────────┼────────────┼───────────────────────────────────────┤
│    332 │ (SWC-000) Unknown │ Medium     │ Function could be marked as external. │
├────────┼───────────────────┼────────────┼───────────────────────────────────────┤
│    353 │ (SWC-000) Unknown │ Medium     │ Function could be marked as external. │
╘════════╧═══════════════════╧════════════╧═══════════════════════════════════════╛

Report for contracts/token/RewardsDistributor.sol
╒════════╤═════════════════════════════════════════════╤════════════╤═══════════════════════════════════════╕
│   Line │ SWC Title                                   │ Severity   │ Short Description                     │
╞════════╪═════════════════════════════════════════════╪════════════╪═══════════════════════════════════════╡
│     94 │ (SWC-108) State Variable Default Visibility │ Low        │ State variable visibility is not set. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    105 │ (SWC-108) State Variable Default Visibility │ Low        │ State variable visibility is not set. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    106 │ (SWC-108) State Variable Default Visibility │ Low        │ State variable visibility is not set. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    108 │ (SWC-108) State Variable Default Visibility │ Low        │ State variable visibility is not set. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    121 │ (SWC-100) Function Default Visibility       │ Low        │ Function visibility is not set.       │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    129 │ (SWC-000) Unknown                           │ Medium     │ Function could be marked as external. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    159 │ (SWC-000) Unknown                           │ Medium     │ Function could be marked as external. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    178 │ (SWC-000) Unknown                           │ Medium     │ Function could be marked as external. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    265 │ (SWC-000) Unknown                           │ Medium     │ Function could be marked as external. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    269 │ (SWC-000) Unknown                           │ Medium     │ Function could be marked as external. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    273 │ (SWC-000) Unknown                           │ Medium     │ Function could be marked as external. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    277 │ (SWC-000) Unknown                           │ Medium     │ Function could be marked as external. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    281 │ (SWC-000) Unknown                           │ Medium     │ Function could be marked as external. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    289 │ (SWC-000) Unknown                           │ Medium     │ Function could be marked as external. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    304 │ (SWC-000) Unknown                           │ Medium     │ Function could be marked as external. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    337 │ (SWC-000) Unknown                           │ Medium     │ Function could be marked as external. │
├────────┼─────────────────────────────────────────────┼────────────┼───────────────────────────────────────┤
│    357 │ (SWC-000) Unknown                           │ Medium     │ Function could be marked as external. │
╘════════╧═════════════════════════════════════════════╧════════════╧═══════════════════════════════════════╛

```
