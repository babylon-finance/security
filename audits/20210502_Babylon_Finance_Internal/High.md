# High

1. // R [ISSUE][HIGH] When we make a Garden Public no more deposits will be allowed because the following require 
Refs: Deposit() in Garden.sol - line 311 and → makeGardenPublic() line 504
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/314)
    
    </aside>
    
    ```jsx
    function deposit(
            uint256 _reserveAssetQuantity,
            uint256 _minGardenTokenReceiveQuantity,
            address _to
        ) public payable override nonReentrant {
            _onlyActive();
            // R [ISSUE][HIGH] When we make a Garden Public no more deposits will be allowed because the following require (see line 488)
            **_require(
                guestListEnabled &&
                    IIshtarGate(IBabController(controller).ishtarGate()).canJoinAGarden(address(this), msg.sender),
                Errors.USER_CANNOT_JOIN
            );**
    ```
    
    ```jsx
    function makeGardenPublic() external override {
            _require(msg.sender == creator, Errors.ONLY_CREATOR);
            _require(guestListEnabled && IBabController(controller).allowPublicGardens(), Errors.GARDEN_ALREADY_PUBLIC);
            // R [ISSUE][HIGH] Be careful, guestListEnabled = false means that no longer deposits will be allowed -> fix deposit require.
            **guestListEnabled = false;**
        }
    ```
    
2. // R [ISSUE][HIGH] We should check that gardenAccessCount[_garden] < maxNumberOfInvites before a new user is set-up, if not we will stuck the modifier "onlyGardenCreator" especially in the batch process (grantGardenAccessBatch) 
(Ref. IstharGate.sol line 282)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/315)
    
    </aside>
    
    ```jsx
    function _setIndividualGardenAccess(
            address _user,
            address _garden,
            uint8 _permission
        ) private returns (uint256) {
            // R [ISSUE][INFORMATIVE] It is recommended to use a require to always have permissions "<= 3" avoiding arbitrary numbers beyond that
            uint256 newItemId = _createOrGetGateNFT(_user);
            if (_permission > 0 && permissionsByCommunity[_garden][_user] == 0) {
                // R [ISSUE][HIGH] We should check that gardenAccessCount[_garden] < maxNumberOfInvites before a new user is set-up, if not we will stuck the modifier "onlyGardenCreator" especially in the batch process (grantGardenAccessBatch)
                **gardenAccessCount[_garden] = gardenAccessCount[_garden].add(1);**
            }
    ```
    
    ```jsx
    modifier onlyGardenCreator(address _garden) {
            require(address(_garden) != address(0), 'Garden must exist');
            IGarden garden = IGarden(_garden);
            require(garden.controller() == address(controller), 'Controller must match');
            // R [ISSUE][MEDIUM] We should always check that the garden is an official garden in the official controller (check that is also the official controller) to avoid malicious gardens and or controllers
            require(msg.sender == garden.creator(), 'Only creator can give access to garden');
            // R [ISSUE][HIGH]  Why not "<=" instead? Be careful as this modifier can stop working just because a batch process is not alerting that we are setting up more users than the max
            **require(gardenAccessCount[_garden] < maxNumberOfInvites, 'The number of contributors must be below the limit');**
            _;
        }
    ```
    
3. // R [ISSUE][HIGH] Check if we are adding more users than max number of users and remove the "<" check in deposit mainthread  as it would also stuck deposits.
(Refs. Garden.sol lines 1019 and 770)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/319)
    
    </aside>
    
    ```jsx
    function _updateContributorDepositInfo(address _contributor, uint256 previousBalance) private {
            Contributor storage contributor = contributors[_contributor];
            // If new contributor, create one, increment count, and set the current TS
            if (previousBalance == 0 || contributor.initialDepositAt == 0) {
                // R [ISSUE][HIGH] Check if we are adding more users than max number of users and remove the "<" check in deposit mainthread
                **totalContributors = totalContributors.add(1);**
                contributor.initialDepositAt = block.timestamp;
            }
    ```
    
    ```jsx
    function deposit(
            uint256 _reserveAssetQuantity,
            uint256 _minGardenTokenReceiveQuantity,
            address _to
        ) public payable override nonReentrant {
            _onlyActive();
            // R [ISSUE][HIGH] When we make a Garden Public no more deposits will be allowed because the following require (see line 488)
            _require(
                guestListEnabled &&
                    IIshtarGate(IBabController(controller).ishtarGate()).canJoinAGarden(address(this), msg.sender),
                Errors.USER_CANNOT_JOIN
            );
            _require(msg.value >= minContribution, Errors.MIN_CONTRIBUTION);
            // if deposit limit is 0, then there is no deposit limit
            if (maxDepositLimit > 0) {
                _require(principal.add(msg.value) <= maxDepositLimit, Errors.MAX_DEPOSIT_LIMIT);
            }
            **// R [ISSUE][HIGH]: Change "<" by "<=" because when the contributors == maxContributors no one would be able to deposit. At the same time it is better to have this check somewhere else to avoid having totalcontributors = maxcontributors + 1. e.g.  "_updateContributorDepositInfo" before adding a new contributor, it will be check during mint.
            _require(totalContributors < maxContributors, Errors.MAX_CONTRIBUTORS);**
    ```
    
4. // R [ISSUE][HIGH] We need to check what access process will have in the future in case Ishtar Gate vs. automation (open publicly to anyone) vs. other contributor checks (reputation, etc.). By using this require, we might need to continue minting any IshtarGate to any new user that maybe is not the best case for the future automated way.
(Refs. BabController.sol line 197)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/321)
    
    </aside>
    
    ```jsx
    function createGarden(
            address _reserveAsset,
            string memory _name,
            string memory _symbol,
            string memory _tokenURI,
            uint256 _seed,
            uint256[] calldata _gardenParams
        ) external payable override returns (address) {
            require(defaultTradeIntegration != address(0), 'Need a default trade integration');
            require(enabledOperations.length > 0, 'Need operations enabled');
            **// R [ISSUE][HIGH] We need to check what access process will have in the future in case Ishtar Gate vs. automation vs. other contributor checks (reputation, etc.)
            require(IIshtarGate(ishtarGate).canCreate(msg.sender), 'User does not have creation permissions');**
            address newGarden =
                IGardenFactory(gardenFactory).createGarden{value: msg.value}(
                    _reserveAsset,
                    address(this),
                    msg.sender,
                    _name,
                    _symbol,
                    _tokenURI,
                    _seed,
                    _gardenParams
                );
    ```
    
5. // R [ISSUE][HIGH] Anyone can execute paykeeper to pay the keeper at arbitrary times reducing garden balance without consent (e.g. DoS). **Credits to Igor.**
(Refs. Garden.sol lines 466)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/307)
    
    </aside>
    
    ```jsx
    function payKeeper(address payable _keeper, uint256 _fee) external override {
            **// R [ISSUE][HIGH] Anyone can execute paykeeper to pay the keeper at arbitrary times reducing garden balance without consent (e.g. DoS)**
            _require(IBabController(controller).isValidKeeper(_keeper), Errors.ONLY_KEEPER);
            keeperDebt = keeperDebt.add(_fee);
            // Pay Keeper in WETH
            // R [ISSUE][INFORMATIVE] Pending TODO in the code under TOOD maybe not found after a quick TODO check.
            // R [ISSUE][MEDIUM] Reserve asset might not be WETH so there is a need to assure compatibility
            // TOOD: Update principal
            // TOOD: Reserve asset may be not WETH
            if (keeperDebt > 0 && IERC20(reserveAsset).balanceOf(address(this)) >= keeperDebt) {
                IERC20(reserveAsset).safeTransfer(_keeper, keeperDebt);
                principal = principal.sub(keeperDebt);
                keeperDebt = 0;
            }
        }
    ```
    
6. // R [ISSUE][HIGH] If we call it more than once (e.g. 1st from a strategy and 2nd or even more by controller) we will create a mess duplicating reserveAssetRewards and withdrawaling twice.
(Refs. Garden.sol lines 420)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/320) but downgraded to LOW due to the exploit used showed that 
    "strategies = strategies.remove(_strategy);" 
    saved last minute the mess ;) of duplicating all withdrawals. Anyway it was fixed to disable non-official strategy calls.
    
    </aside>
    
    ```jsx
    /**
         * When an strategy finishes execution, we want to make that eth available for withdrawals
         * from members of the garden.
         *
         * @param _amount                        Amount of WETH to convert to ETH to set aside until the window ends
         * @param _rewards                       Amount of WETH to convert to ETH to set aside forever
         * @param _returns                       Profits or losses that the strategy received
         */
        function startWithdrawalWindow(
            uint256 _amount,
            uint256 _rewards,
            int256 _returns,
            address _strategy
        **) external override {
            // R [ISSUE][HIGH] If we call more it more than once (e.g. 1st from a strategy and 2nd or even more by controller) we will create a mess duplicating reserveAssetRewards and withdrawaling twice
            _require(
                (strategyMapping[msg.sender] && address(IStrategy(msg.sender).garden()) == address(this)) ||
                    msg.sender == controller,
                Errors.ONLY_STRATEGY_OR_CONTROLLER
            );**
            // Updates reserve asset
            principal = principal.toInt256().add(_returns).toUint256();
            if (withdrawalsOpenUntil > block.timestamp) {
                withdrawalsOpenUntil = block.timestamp.add(
                    withdrawalWindowAfterStrategyCompletes.sub(withdrawalsOpenUntil.sub(block.timestamp))
                );
            } else {
                withdrawalsOpenUntil = block.timestamp.add(withdrawalWindowAfterStrategyCompletes);
            }
            **reserveAssetRewardsSetAside = reserveAssetRewardsSetAside.add(_rewards);
            reserveAssetPrincipalWindow = reserveAssetPrincipalWindow.add(_amount);
            // Both are converted to weth
            IWETH(WETH).withdraw(_amount.add(_rewards));**
    
            // Mark strategy as finalized
            absoluteReturns.add(_returns);
            strategies = strategies.remove(_strategy);
            finalizedStrategies.push(_strategy);
            strategyMapping[_strategy] = false;
        }
    ```
    
7. // R [ISSUE][HIGH] First param seems to substract in a wrong way 2xprotocolProfits as they are included in profits.
(Refs. Strategy.sol line 859)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/312)
    
    </aside>
    
    ```jsx
    function _transferStrategyPrincipal(uint256 _fee) internal {
            //notion // R [ISSUE][LOW] The chosen fee has a direct impact on the capital returned. As there are several implications around capital returned, expected return, etc. if the fee is too high, people might complaint 
            //notion // R [ISSUE][MEDIUM] If reserve asset is not WETH but USDC (6 decimals) the fee should be using 6 decimals as well.
            capitalReturned = IERC20(garden.reserveAsset()).balanceOf(address(this)).sub(_fee);
            address reserveAsset = garden.reserveAsset();
            int256 reserveAssetDelta = capitalReturned.toInt256().sub(capitalAllocated.toInt256());
            uint256 protocolProfits = 0;
            // Strategy returns were positive
            uint256 profits = capitalReturned > capitalAllocated ? capitalReturned.sub(capitalAllocated) : 0; // in reserve asset (weth)
            if (capitalReturned >= capitalAllocated) {
                // Send weth performance fee to the protocol
                protocolProfits = IBabController(controller).protocolPerformanceFee().preciseMul(profits);
                IERC20(reserveAsset).safeTransferFrom(
                    address(this),
                    IBabController(controller).treasury(),
                    protocolProfits
                );
                reserveAssetDelta.add(int256(-protocolProfits));
            } else {
                // Returns were negative
                // Burn strategist stake and add the amount to the garden
                uint256 burningAmount =
                    (stake.sub(capitalReturned.preciseDiv(capitalAllocated).preciseMul(stake))).multiplyDecimal(
                        STAKE_QUADRATIC_PENALTY_FOR_LOSSES
                    );
                if (IERC20(address(garden)).balanceOf(strategist) < burningAmount) {
                    // Avoid underflow burning more than its balance
                    burningAmount = IERC20(address(garden)).balanceOf(strategist);
                }
    
                garden.burnStrategistStake(strategist, burningAmount);
                // R [ISSUE][MEDIUM] We need to confirm that reserveAssetDelta (reserveAsset) and burningAmount (garden ERC20) are using the same decimals
                reserveAssetDelta.add(int256(burningAmount));
            }
            // Return the balance back to the garden
            IERC20(reserveAsset).safeTransferFrom(address(this), address(garden), capitalReturned.sub(protocolProfits));
            // Start a redemption window in the garden with the capital plus the profits for the lps
            (, , uint256 lpsProfitSharing) = IBabController(controller).getProfitSharing();
            garden.startWithdrawalWindow(
                **// R [ISSUE][CHECK] First param seems to substract in a wrong way 2xprotocolProfits as they are included in profits.
                capitalReturned.sub(protocolProfits).sub(profits).add((profits).preciseMul(lpsProfitSharing)), //capital returned - 20% - 5% !!**
                profits.sub(profits.preciseMul(lpsProfitSharing)).sub(protocolProfits), // 15%
                reserveAssetDelta,
                address(this)
            );
    ```
    
8. // R [ISSUE][HIGH] This statement is not adding or reducing protocolProfits from reserveAsset Delta
(Refs. Strategy.sol line 826 and 844)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/312)
    
    </aside>
    
    ```jsx
    function _transferStrategyPrincipal(uint256 _fee) internal {
            capitalReturned = IERC20(garden.reserveAsset()).balanceOf(address(this)).sub(_fee);
            console.log('Capital Returned before fee', capitalReturned.add(_fee));
            address reserveAsset = garden.reserveAsset();
            int256 reserveAssetDelta = capitalReturned.toInt256().sub(capitalAllocated.toInt256());
            console.log('Reserve Asset Delta before', Math.abs(reserveAssetDelta).toUint256());
            uint256 protocolProfits = 0;
            // Strategy returns were positive
            uint256 profits = capitalReturned > capitalAllocated ? capitalReturned.sub(capitalAllocated) : 0; // in reserve asset (weth)
            if (capitalReturned >= capitalAllocated) {
                // Send weth performance fee to the protocol
                protocolProfits = IBabController(controller).protocolPerformanceFee().preciseMul(profits);
                IERC20(reserveAsset).safeTransferFrom(
                    address(this),
                    IBabController(controller).treasury(),
                    protocolProfits
                );
                console.log('TARGET profit PROTOCOL PROFITS', protocolProfits);
                console.log('Reserve Asset Delta just before profits', Math.abs(reserveAssetDelta).toUint256());
                **// R [ISSUE][HIGH] This statement is not adding or reducing protocolProfits from reserveAsset Delta
                reserveAssetDelta.add(int256(-protocolProfits));**
                console.log('Reserve Asset Delta after profits', Math.abs(reserveAssetDelta).toUint256());
            } else {
                // Returns were negative
                // Burn strategist stake and add the amount to the garden
                uint256 burningAmount =
                    (stake.sub(capitalReturned.preciseDiv(capitalAllocated).preciseMul(stake))).multiplyDecimal(
                        STAKE_QUADRATIC_PENALTY_FOR_LOSSES
                    );
                if (IERC20(address(garden)).balanceOf(strategist) < burningAmount) {
                    // Avoid underflow burning more than its balance
                    burningAmount = IERC20(address(garden)).balanceOf(strategist);
                }
    
                garden.burnStrategistStake(strategist, burningAmount);
                console.log('TARGET no profit PROTOCOL PROFITS', protocolProfits);
                console.log('Reserve Asset Delta just before profits', Math.abs(reserveAssetDelta).toUint256());
                **// R [ISSUE][HIGH] This statement is not adding or reducing protocolProfits from reserveAsset Delta
                reserveAssetDelta.add(int256(burningAmount));**
                console.log('Reserve Asset Delta after burning (no profit)', Math.abs(reserveAssetDelta).toUint256());
            
            // Return the balance back to the garden
            IERC20(reserveAsset).safeTransferFrom(address(this), address(garden), capitalReturned.sub(protocolProfits));
            // Start a redemption window in the garden with the capital plus the profits for the lps
            (, , uint256 lpsProfitSharing) = IBabController(controller).getProfitSharing();
    ```
    
9. // R [ISSUE][HIGH] Calculation is not properly adjusting the supply pro-rata to the execution window as _startingQuarter (means number of quarter e.g. 1) is different from _startingSlot (timestamp of the start of the slot  (≠ from quarter number)). Also the window should be EPOCH_DURATION instead of block.timestamp.sub(_startingQuarter) to calculate the timePercentage of the execution out of the total execution. As a result, the whole supply of a quarter was multiplied by the strategypower/protocolpower ratio instead of just the execution pro-rata window, more tokens (> than its real execution time deserved's), were assigned to the strategy.
(Refs. RewardsDistributor.sol line 961)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    /**
         * Check the strategy rewards for strategies starting and ending in the same quarter
         * @param _strategy         Strategy
         * @param _startingQuarter  Starting quarter
         */
        function _getStrategyRewardsOneQuarter(address _strategy, uint256 _startingQuarter)
            private
            view
            onlyMiningActive
            returns (uint256)
        {
            IStrategy strategy = IStrategy(_strategy);
            **// R [ISSUE][HIGH] Calculation is not properly adjusting the supply pro-rata to the execution window as _startingQuarter (means number of quarter e.g. 1) which is different from _startingSlot (timestamp of the start of the slot). Also the window should be EPOCH_DURATION instead of block.timestamp.sub(_startingQuarter).**
            uint256 strategyOverTime =
                strategy.capitalAllocated().mul(strategy.exitedAt().sub(strategy.executedAt())).sub(
                    strategy.rewardsTotalOverhead()
                );
            return
                strategyOverTime
                    .preciseDiv(protocolPerQuarter[_startingQuarter].quarterPower)
                    .preciseMul(uint256(protocolPerQuarter[_startingQuarter].supplyPerQuarter))
                    **.mul(strategy.exitedAt().sub(_startingQuarter))
                    .div(block.timestamp.sub(_startingQuarter));
        }**
    ```
    
10. // R [ISSUE][HIGH] Edges bounds are not handled properly for strategies during less than a quarter but going through 2 different consecutive quarters. As a result it was trying to provide more tokens than deserved however an overflow supply require protected from that but as trade-off, it resulted in getRewards being stucked and not working.
(Refs. RewardsDistributor.sol line 1297)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    /**
         * Calculates the range (starting quarter and ending quarter since START_TIME)
         * @param _from   Starting timestamp
         * @param _to     Ending timestamp
         */
        function _getRewardsWindow(uint256 _from, uint256 _to) internal view returns (uint256, uint256) {
            **// R [ISSUE][HIGH] Edges bounds are not handled properly for strategies during less than a quarter but going through 2 different quarters.
            uint256 quarters = (_to.sub(_from).preciseDivCeil(EPOCH_DURATION)).div(1e18);
            uint256 startingQuarter = (_from.sub(START_TIME).preciseDivCeil(EPOCH_DURATION)).div(1e18);
            return (quarters.add(1), startingQuarter.add(1));**
        }
    }
    ```
    
11. // R [ISSUE][HIGH] ReserveAsset/DAI pair price change might produce substraction underflow along the time if pair price goes down (get more DAI for the same quantity of reserveAsset tokens). This underflow might happen at the strategy level but also at the protocol level. On the other hand if price goes up (less DAI for the same amount of reserveAsset tokens) we might be reducing less than expected/needed. We need to take care of the price variation in all possible cases and the variation of each strategy might be the same at protocol level. For example, after a strategy finishes, its contribution to the protocol power must be reset to zero. If not, other running strategies might not have right % power associated to their rewards calculations.
(Refs. RewardsDistributor.sol line 475)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/351)
    
    </aside>
    
    ```jsx
    function _updateProtocolPrincipal(
            address _strategy,
            uint256 _capital,
            bool _addOrSubstract
        ) internal {
            IStrategy strategy = IStrategy(_strategy);
            **// Normalizing into DAI
            IPriceOracle oracle = IPriceOracle(IBabController(controller).priceOracle());
            uint256 pricePerTokenUnit = oracle.getPrice(IGarden(strategy.garden()).reserveAsset(), DAI);
            _capital = _capital.preciseMul(pricePerTokenUnit);**
            ProtocolPerTimestamp storage protocolCheckpoint = protocolPerTimestamp[block.timestamp];
            if (_addOrSubstract == false) {
                // Substract
                console.log('Before Protocol substract', protocolPrincipal, _capital);
                **protocolPrincipal = protocolPrincipal.sub(_capital);**
                console.log('After Protocol substract', protocolPrincipal);
            } else {
                console.log('Before Protocol add', protocolPrincipal, _capital);
                protocolPrincipal = protocolPrincipal.add(_capital);
                console.log('After Protocol add ', protocolPrincipal);
            }
            protocolCheckpoint.principal = protocolPrincipal;
            protocolCheckpoint.time = block.timestamp;
            protocolCheckpoint.quarterBelonging = _getQuarter(block.timestamp);
            protocolCheckpoint.timeListPointer = pid;
            if (pid == 0) {
                // The very first strategy of all strategies in the mining program
                protocolCheckpoint.power = 0;
            } else {
                // Any other strategy different from the very first one (will have an antecesor)
                protocolCheckpoint.power = protocolPerTimestamp[timeList[pid.sub(1)]].power.add(
                    protocolCheckpoint.time.sub(protocolPerTimestamp[timeList[pid.sub(1)]].time).mul(
                        protocolPerTimestamp[timeList[pid.sub(1)]].principal
                    )
                );
            }
    ```
    
12. // R [ISSUE][HIGH] Strategy power calculation based only on capitalAllocated at the end of the strategy execution (whatever it is)  + a variable (named rewardsTotalOverhead) to keep control of potential overhead) does not correctly support unwind strategy behavior. _updatePowerOverhead is always executed in _updateProtocolPrincipal either for adding or substracting at addProtocolPrincipal and substractProtocolPrincipal respectively. It really worked well as Overhead was not really added when substracting capital (unwind) but it was trying to do so (which is a risk as it only has to add overhead if increasing capital to the strategy not viceversa). It was not added because updatedAt equals block.timestamp in that moment so its substraction was 0 which multiplied by the capital (added or substracted) it was again 0, but it tried to add anyway which is not good). As this measure/result was not intentional or a countermeasure supporting unwind, a re-design of strategy reward calculation has to be done to support unwind strategies. The re-design alleviate finalizeStrategy tx costs as quarter checkpoints have to be implemented along each re-allocation instead, it is to keep track on allocation changes with accuracy along the time (either adding or substracting capital). As a result it was also providing less BABL Rewards only in case of unwinding strategies.
(Refs. RewardsDistributor.sol line 269, 501, 895, 936, 980)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/351)
    
    </aside>
    
    ```jsx
    
        /**
         * Function that removes the capital received to the total principal of the protocol per timestamp
         * @param _capital                Amount of capital in any type of asset to be normalized into DAI
         */
        function substractProtocolPrincipal(uint256 _capital) external override onlyStrategy onlyMiningActive {
            IStrategy strategy = IStrategy(msg.sender);
            if (strategy.enteredAt() >= START_TIME) {
                // onlyMiningActive control, it does not create a checkpoint if the strategy is not part of the Mining Program
                **_updateProtocolPrincipal(address(strategy), _capital, false);**
            }
        }
    ```
    
    ```jsx
    
    function _updateProtocolPrincipal(
            address _strategy,
            uint256 _capital,
            bool _addOrSubstract
        ) internal {
    [...]
    				timeList.push(block.timestamp); // Register of added strategies timestamps in the array for iteration
            // Here we control the accumulated protocol power per each quarter
            // Create the quarter checkpoint in case the checkpoint is the first in the epoch
            _addProtocolPerQuarter(block.timestamp);
            // We update the rewards overhead if any in normalized DAI
            **_updatePowerOverhead(strategy, _capital);**
            pid++;
    ```
    
    ```jsx
    * function _updatePowerOverhead(IStrategy _strategy, uint256 _capital) private onlyMiningActive {
            if (_strategy.updatedAt() != 0) {
                // There will be overhead after the first execution not before
                if (_getQuarter(block.timestamp) == _getQuarter(_strategy.updatedAt())) {
                    // The overhead will remain within the same epoch
                    rewardsPowerOverhead[address(_strategy)][_getQuarter(block.timestamp)] = rewardsPowerOverhead[
                        address(_strategy)
                    ][_getQuarter(block.timestamp)]
                        **.add(_capital.mul(block.timestamp.sub(_strategy.updatedAt())));**
                        console.log('ADDED OVERHEAD %s - quarter %s', rewardsPowerOverhead[address(_strategy)][_getQuarter(block.timestamp)], _getQuarter(block.timestamp));
    
                } else {
                    // We need to iterate since last update of the strategy capital
                    (uint256 numQuarters, uint256 startingQuarter) =
                        _getRewardsWindow(_strategy.updatedAt(), block.timestamp);
                    uint256 overheadPerQuarter = _capital.mul(**block.timestamp.sub(_strategy.updatedAt()))**.div(numQuarters);
                    for (uint256 i = 0; i <= numQuarters.sub(1); i++) {
                        rewardsPowerOverhead[address(_strategy)][startingQuarter.add(i)] = rewardsPowerOverhead[
                            address(_strategy)
                        ][startingQuarter.add(i)]
                            **.add(overheadPerQuarter);**
                        console.log('ADDED OVERHEAD %s - quarter %s', rewardsPowerOverhead[address(_strategy)][startingQuarter.add(i)], startingQuarter.add(i));
                    }
                }
            }
        }
    ```
    
    ```jsx
    function getStrategyRewards(address _strategy) external view override returns (uint96) {
            IStrategy strategy = IStrategy(_strategy);
            _require(strategy.exitedAt() != 0, Errors.STRATEGY_IS_NOT_OVER_YET);
            IPriceOracle oracle = IPriceOracle(IBabController(controller).priceOracle());
            uint256 pricePerTokenUnit = oracle.getPrice(IGarden(strategy.garden()).reserveAsset(), DAI);
            **uint256 allocated = strategy.capitalAllocated().preciseMul(pricePerTokenUnit);**
            uint256 returned = strategy.capitalReturned().preciseMul(pricePerTokenUnit);
            if ((strategy.enteredAt() >= START_TIME) && (START_TIME != 0)) {
                // We avoid gas consuming once a strategy got its BABL rewards during its finalization
                uint256 rewards = strategy.strategyRewards();
                if (rewards != 0) {
                    return Safe3296.safe96(rewards, 'overflow 96 bits');
                }
                // If the calculation was not done earlier we go for it
                (uint256 numQuarters, uint256 startingQuarter) =
                    _getRewardsWindow(strategy.executedAt(), strategy.exitedAt());
                uint256 bablRewards = 0;
                if (numQuarters <= 1) {
                    bablRewards = _getStrategyRewardsOneQuarter(_strategy, allocated, startingQuarter); // Proportional supply till that moment within the same epoch
                    _require(
                        bablRewards <= protocolPerQuarter[startingQuarter].supplyPerQuarter,
                        Errors.OVERFLOW_IN_SUPPLY
                    );
                    _require(
                        allocated.mul(strategy.exitedAt().sub(strategy.executedAt())).sub(
                            strategy.rewardsTotalOverhead()
                        ) <= protocolPerQuarter[startingQuarter].quarterPower,
                        Errors.OVERFLOW_IN_POWER
                    );
                } else {
                    bablRewards = _getStrategyRewardsSomeQuarters(_strategy, allocated, startingQuarter, numQuarters);
                }
    
                // Babl rewards will be proportional to the total return (profit) with a max cap of x2
                uint256 percentageMul = returned.preciseDiv(allocated);
                if (percentageMul > 2e18) percentageMul = 2e18;
                bablRewards = bablRewards.preciseMul(percentageMul);
                return Safe3296.safe96(bablRewards, 'overflow 96 bits');
            } else {
                return 0;
            }
        }
    ```
    
13. // R [ISSUE][HIGH] A reserveAsset profit inject into a strategy might impact rewards (e.g. BABL) as the returning amount is considered part of the strategy profits. The x2 cap was working to limit damages but malicious capital should not be counted.
(Refs. RewardsDistributor.sol line 269, 501, 895, 936, 980)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/370)
    
    </aside>
    
    ```jsx
    
        function _transferStrategyPrincipal() internal {
            **capitalReturned = IERC20(garden.reserveAsset()).balanceOf(address(this));**
    ```