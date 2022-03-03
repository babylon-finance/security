# Low

1. // R [ISSUE][LOW] Check that _user is not the garden creator to avoid removing itself
(Refs. IshtarGate.sol line 127)
    
    <aside>
    💡 BY DESIGN (can recover anytime)
    
    </aside>
    
    ```jsx
    function setGardenAccess(
            address _user,
            address _garden,
            uint8 _permission
        ) external override onlyGardenCreator(_garden) returns (uint256) {
            require(address(_user) != address(0), 'User must exist');
            //notion // R [ISSUE][CHECK] Check what happens if the garden creator cancel some user access and if they are active users (with deposits) what happens to their deposits, votes... is it maintained somehow e.g. to re-use its position for a real active user (new one)
            **// R [ISSUE][LOW] Check that _user is not the garden creator to avoid itself
            return _setIndividualGardenAccess(_user, _garden, _permission);**
        }
    ```
    
2. // R [ISSUE][LOW] What happens if a creator is disabled (-> cannot create more) but while being malicious it gave several permissions to other malicious strategists? What happens to this garden and its users malicious or not? Maybe a way (function) to get the list of users with access and their access is recommended.
(Refs. IshtarGate.sol line 167)
    
    <aside>
    💡 ACCEPTED AS PART OF DESIGN
    
    </aside>
    
    ```jsx
    /**
         * Awards the ishtar gate to a user and give/remove him garden creation capabilities.
         *
         * @param _user               Address of the user
         * @param _canCreate          Boolean with permissions as to whether the user can create gardens
         */
        function setCreatorPermissions(address _user, bool _canCreate) external override onlyOwner returns (uint256) {
            **// R [ISSUE][LOW] What happens if a creator is disabled (-> cannot create more) but while being malicious it gave several permissions to other malicious strategists? What happens to this garden and its users malicious or not? Maybe a way (function) to get the list of users with access and their access is recommended.** 
            return _setCreatorPermissions(_user, _canCreate);
        }
    ```
    
3. // R [ISSUE][LOW] It is recommended to give a maximum number of gardens by default, not infinite
(Refs. IshtarGate.sol line 300)
    
    <aside>
    💡 ACCEPTED AS PART OF DESIGN
    
    </aside>
    
    ```jsx
    function _setCreatorPermissions(address _user, bool _canCreate) private returns (uint256) {
            **// R [ISSUE][LOW] It is recommended to give a maximum number of gardens by default, not infinite**
            require(address(_user) != address(0), 'User must exist');
            uint256 newItemId = _createOrGetGateNFT(_user);
            canCreateAGarden[_user] = _canCreate;
            emit GardenCreationPower(_user, _canCreate, newItemId);
            return newItemId;
        }
    ```
    
4. // R [ISSUE][LOW]: This mint creator-creator, msg.value-msg.value need to be checked to understand duplicated params (from=to ?? vs. address(this)).
(Refs. Garden.sol line 231)
    
    <aside>
    💡 ACCEPTED AS PART OF DESIGN
    
    </aside>
    
    ```jsx
    _gardenParams[4],
                _gardenParams[5],
                _gardenParams[6],
                _gardenParams[7],
                _gardenParams[8]
            );
            active = true;
    
            // Deposit
            IWETH(WETH).deposit{value: msg.value}();
            // R [CHECK][INFORMATIVE]: Is this transfer of tokens enabled by default? What about the rest? Check in BabController.
            **// R [ISSUE][LOW]: This mint creator-creator, msg.value-msg.value need to be checked to understand duplicated params (from=to ?? vs. address(this)). 
            _mintGardenTokens(creator, creator, msg.value, msg.value, 0);**
        }
    ```
    
5. // R [ISSUE][LOW]: Check if using block.timestamp or block.number
(Refs. Garden.sol line 272)
    
    <aside>
    💡 AGREED THAT IS ENOUGH BY TIMESTAMP DIFF > 0
    
    </aside>
    
    ```jsx
    function _start(
            uint256 _creatorDeposit,
            uint256 _maxDepositLimit,
            uint256 _minGardenTokenSupply,
            uint256 _minLiquidityAsset,
            uint256 _depositHardlock,
            uint256 _minContribution,
            uint256 _strategyCooldownPeriod,
            uint256 _minVotersQuorum,
            uint256 _minStrategyDuration,
            uint256 _maxStrategyDuration
        ) private {
            _require(_minContribution >= ABSOLUTE_MIN_CONTRIBUTION, Errors.MIN_CONTRIBUTION);
            _require(_creatorDeposit >= _minContribution, Errors.MIN_CONTRIBUTION);
            _require(_creatorDeposit >= _minGardenTokenSupply, Errors.MIN_LIQUIDITY);
            _require(_creatorDeposit <= _maxDepositLimit, Errors.MAX_DEPOSIT_LIMIT);
            _require(_maxDepositLimit <= MAX_DEPOSITS_FUND_V1, Errors.MAX_DEPOSIT_LIMIT);
            IBabController babController = IBabController(controller);
            _require(_minGardenTokenSupply > 0, Errors.MIN_TOKEN_SUPPLY);
            **// R [ISSUE][LOW]: Is enough just being depositHardlock above 0?
            _require(_depositHardlock > 0, Errors.DEPOSIT_HARDLOCK);**
    ```
    
6. // R [ISSUE][LOW]: check msg.sender as "mint from" (should be the user isnt it?) and check why we use _to instead of msg.sender if we want to be sure that no typos/mistakes are done in receiving address.
(Refs. Garden.sol line 345)
    
    <aside>
    💡 VERIFIED AS PART OF THE RESERVE ASSETS
    
    </aside>
    
    ```jsx
    function deposit(
            uint256 _reserveAssetQuantity,
            uint256 _minGardenTokenReceiveQuantity,
            address _to
        ) public payable override nonReentrant {
            _onlyActive();
            //notion // R [ISSUE][HIGH] When we make a Garden Public no more deposits will be allowed because the following require (see line 488)
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
            //notion // R [ISSUE][HIGH]: Change "<" by "<=" because when the contributors == maxContributors no one would be able to deposit. At the same time it is better to have this check somewhere else to avoid having totalcontributors = maxcontributors + 1. e.g.  "_updateContributorDepositInfo" before adding a new contributor, it will be check during mint.
            _require(totalContributors < maxContributors, Errors.MAX_CONTRIBUTORS);
            _require(msg.value == _reserveAssetQuantity, Errors.MSG_VALUE_DO_NOT_MATCH);
            // Always wrap to WETH
            IWETH(WETH).deposit{value: msg.value}();
            // Check this here to avoid having relayers
            reenableEthForStrategies();
    
            _validateReserveAsset(reserveAsset, _reserveAssetQuantity);
    
            (uint256 protocolFees, uint256 netFlowQuantity) = _getFees(_reserveAssetQuantity, true);
    
            // Check that total supply is greater than min supply needed for issuance
            _require(totalSupply() >= minGardenTokenSupply, Errors.MIN_TOKEN_SUPPLY);
    
            // gardenTokenQuantity has to be at least _minGardenTokenReceiveQuantity
            _require(netFlowQuantity >= _minGardenTokenReceiveQuantity, Errors.MIN_TOKEN_SUPPLY);
    
            // Send Protocol Fee
            payProtocolFeeFromGarden(reserveAsset, protocolFees);
    
            // Mint tokens
            **// R [ISSUE][LOW]: check msg.sender as "mint from" (should be the user isnt it?) and check why we use _to instead of msg.sender if we want to be sure that no typos/mistakes are done in receiving address.
            _mintGardenTokens(msg.sender, _to, netFlowQuantity, principal.add(netFlowQuantity), protocolFees);
        }**
    ```
    
7. // R [ISSUE][LOW] Check whether or not this modifier has to be used out of running strategies as it will be false e.g. in withdrawal windows or or burning strategist stake
(Refs. Garden.sol line 770)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    /**
         * Throws if the sender is not an strategy of this garden
         */
         **// R [ISSUE][LOW] Check whether or not this modifier has to be used out of running strategies as it will be false e.g. in withdrawal windows**
        function _onlyStrategy() private view {
            **_require(strategyMapping[msg.sender], Errors.ONLY_STRATEGY);**
        }
    ```
    
8. // R [ISSUE][LOW] It is recommended to check and associate each strategy or strategyNFT to its garden, it is recommended to check who the caller is
(Refs. StrategyNFT.sol line 77)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    function initialize(
            address _controller,
            address _strategy,
            string memory _name,
            string memory _symbol
            **// R [ISSUE][LOW] It is recommended to check and associate each strategy or strategyNFT to its garden, it is recommended to check who the caller is**
        ) external override initializer {
            require(address(_controller) != address(0), 'Controller must exist');
            require(bytes(_name).length < 50, 'Strategy Name is too long');
            __ERC721_init(_name, _symbol);
            controller = IBabController(_controller);
            strategy = IStrategy(_strategy);
        }
    ```
    
9. // R [ISSUE][LOW] The chosen fee has a direct impact on the capital returned. As there are several implications around capital returned, expected return, etc. if the fee is too high due to a wrong gas calculation, people might complain
(Refs. Strategy.sol line 818)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    **function _transferStrategyPrincipal(uint256 _fee) internal {
            // R [ISSUE][LOW] The chosen fee has a direct impact on the capital returned. As there are several implications around capital returned, expected return, etc. if the fee is too high, people might complaint** 
            // R [ISSUE][MEDIUM] If reserve asset is not WETH but USDC (6 decimals) the fee should be using 6 decimals as well.
            **capitalReturned = IERC20(garden.reserveAsset()).balanceOf(address(this)).sub(_fee);**
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
            }
    ```
    
10. // R [ISSUE][LOW] By using changeStrategyDuration a strategist can bypass the minimal duration so a new check is recommended like _require(__newDuration >= garden.minStrategyDuration())
(Refs. Strategy.sol line 519)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    /**
         * Lets the strategist change the duration of the strategy
         * @param _newDuration            New duration of the strategy
         */
        function changeStrategyDuration(uint256 _newDuration) external override onlyStrategist {
            _require(!finalized, Errors.STRATEGY_IS_ALREADY_FINALIZED);
            _require(_newDuration < duration, Errors.DURATION_NEEDS_TO_BE_LESS);
            **// R [ISSUE][LOW] By using changeStrategyDuration a strategist can bypass the minimal duration so a new check is recommended like _require(__newDuration >= garden.minStrategyDuration())**
            emit StrategyDurationChanged(_newDuration, duration);
            **duration = _newDuration;**
        }
    ```
    
11. // R [ISSUE][LOW] we do not want delete distribution[account] be executed by non vested users
(Refs. TimeLockedToken line 330)
    
    <aside>
    💡 SOLVED ([https://github.com/babylon-finance/protocol/pull/329](https://github.com/babylon-finance/protocol/pull/329))
    
    </aside>
    
    ```jsx
    function lockedBalance(address account) public returns (uint256) {
            // get amount from distributions locked tokens (if any)
    
            uint256 lockedAmount = viewLockedBalance(account);
    
            **// R [ISSUE][LOW] we do not want delete distribution[account] be executed by non vested users**
            // in case of vesting has passed, all tokens are now available so we set mapping to 0
            if (block.timestamp >= vestedToken[account].vestingEnd && msg.sender == account && lockedAmount == 0) {
                **delete distribution[account];**
            }
            return lockedAmount;
        }
    ```