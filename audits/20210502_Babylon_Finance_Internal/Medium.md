# Medium

1. // R [ISSUE][MEDIUM] We should always check that the garden is an official garden in the official controller (check that is also the official controller) to avoid malicious gardens and or controllers 
(Refs. IshtarGate.sol line 74)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/315)
    
    </aside>
    
    ```jsx
    modifier onlyGardenCreator(address _garden) {
            require(address(_garden) != address(0), 'Garden must exist');
            IGarden garden = IGarden(_garden);
            require(garden.controller() == address(controller), 'Controller must match');
            **// R [ISSUE][MEDIUM] We should always check that the garden is an official garden in the official controller (check that is also the official controller) to avoid malicious gardens and or controllers**
            require(msg.sender == garden.creator(), 'Only creator can give access to garden');
            // R [ISSUE][HIGH]  Why not "<=" instead? Be careful as this modifier can stop working just because a batch process is not alerting that we are setting up more users than the max
            require(gardenAccessCount[_garden] < maxNumberOfInvites, 'The number of contributors must be below the limit');
            _;
        }
    ```
    
2. // R [ISSUE][MEDIUM] Recommended the usage of NonReentrant 
(Refs. Treasury.sol line 74)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    function sendTreasuryFunds(
            address _asset,
            uint256 _amount,
            address _to
        ) external onlyOwner {
            // R [ISSUE][MEDIUM] Recommended the usage of NonReentrant
            require(_asset != address(0), 'Asset must exist');
            require(_to != address(0), 'Target address must exist');
            IERC20(_asset).safeTransferFrom(address(this), _to, _amount);
            emit TreasuryFundsSent(_asset, _amount, _to);
        }
    ```
    
3. // R [ISSUE][MEDIUM] Recommended the usage of NonReentrant 
(Refs. Treasury.sol line 87)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    function sendTreasuryETH(uint256 _amount, address payable _to) external onlyOwner {
            // R [ISSUE][MEDIUM] Recommended the usage of NonReentrant
            require(_to != address(0), 'Target address must exist');
            require(address(this).balance >= _amount, 'Not enough funds in treasury');
            Address.sendValue(_to, _amount);
            // R [ISSUE][LOW] Check whether from address should be address(0) or address(this)
            emit TreasuryFundsSent(address(0), _amount, _to);
        }
    ```
    
4. // R [ISSUE][MEDIUM] It is recommended to check whether or not the _nftAddress is a systemContract to avoid malicious NFT 
(Refs. Garden.sol line 210 but also in GardenFactory.sol line 69 after the NFT is set)
    
    <aside>
    💡 SOLVED as the new require introduced in GardenFactory disable the possibility to createGardens out of the controller to disable malicious intentions:
    
    require(msg.sender == controller, "Only the controller can create gardens");
    
    </aside>
    
    ```jsx
    function initialize(
            address _reserveAsset,
            address _controller,
            address _creator,
            string memory _name,
            string memory _symbol,
            uint256[] calldata _gardenParams,
            address _nftAddress
            //notion // R [ISSUE][MEDIUM] Check with Jota if there could be specific modifiers for initializing flow (avoiding malicious initializations)
        ) public payable initializer {
            // R [INFORMATIVE]: What about to add a check in the number of gardens to be created per address?
            _require(bytes(_name).length < 50, Errors.NAME_TOO_LONG);
            _require(_creator != address(0), Errors.ADDRESS_IS_ZERO);
            _require(_controller != address(0), Errors.ADDRESS_IS_ZERO);
            _require(_reserveAsset != address(0), Errors.ADDRESS_IS_ZERO);
            _require(_gardenParams.length == 9, Errors.GARDEN_PARAMS_LENGTH);
            _require(IBabController(_controller).isValidReserveAsset(_reserveAsset), Errors.MUST_BE_RESERVE_ASSET);
            __ERC20_init(_name, _symbol);
    
            controller = _controller;
            reserveAsset = _reserveAsset;
            // R [ISSUE][LOW]: it is recommended to identify msg.sender instead of having creator as a param (as we are doing with msg.value) to avoid typos or wrong addresses.
            creator = _creator;
            maxContributors = 100;
            **// R [ISSUE][MEDIUM] It is recommended to check whether or not the _nftAddress is a systemContract to avoid malicious NFT
            nftAddress = _nftAddress;**
            guestListEnabled = true;
    
            _start(
                msg.value,
    ```
    
    ```jsx
    function createGarden(
            address _reserveAsset,
            address _controller,
            address _creator,
            string memory _name,
            string memory _symbol,
            string memory _tokenURI,
            uint256 _seed,
            uint256[] calldata _gardenParams
            //notion // R [ISSUE][MEDIUM] It is recommended to use controller (upgradeable) address for example initializing it in the constructor instead of using it as param each time, it help to avoid mistakes but also to avoid malicious potential behavior
            //notion // R [ISSUE][MEDIUM] If the caller is a system contract, it is recommended to use isSystemContract as modifier to check whether or not the caller is a system contract.
        ) external payable override returns (address) {
            address payable clone = payable(Clones.clone(garden));
            address cloneNFT = Clones.clone(gardenNFT);
            GardenNFT(cloneNFT).initialize(_controller, address(clone), _name, _symbol, _tokenURI, _seed);
            **// R [ISSUE][MEDIUM]: it is recommended to add gardenNFT as a valid systemcontroller to avoid malicious gardenNFT passed as params.**
            Garden(clone).initialize{value: msg.value}(
                _reserveAsset,
                _controller,
                _creator,
                _name,
                _symbol,
                _gardenParams,
                cloneNFT
            );
            return clone;
    ```
    
5. // R [ISSUE][MEDIUM] It is recommended to use controller (upgradeable) address for example initializing it in the constructor instead of using it as param each time, it help to avoid mistakes but also to avoid malicious potential behavior using malicious controllers.
(Refs. [GardenFactory.sol] mitigating risks but also at GardenNFT.sol, Garden.sol, Strategy.sol, StrategyFactory.sol)
    
    <aside>
    💡 SOLVED (Set controller at factories PR#308)
    
    </aside>
    
    ```jsx
    function createGarden(
            address _reserveAsset,
            address _controller,
            address _creator,
            string memory _name,
            string memory _symbol,
            string memory _tokenURI,
            uint256 _seed,
            uint256[] calldata _gardenParams
            **// R [ISSUE][MEDIUM] It is recommended to use controller (upgradeable) address for example initializing it in the constructor instead of using it as param each time, it help to avoid mistakes but also to avoid malicious potential behavior**
            // R [ISSUE][MEDIUM] If the caller is a system contract, it is recommended to use isSystemContract as modifier to check whether or not the caller is a system contract.
        ) external payable override returns (address) {
            address payable clone = payable(Clones.clone(garden));
            address cloneNFT = Clones.clone(gardenNFT);
            **GardenNFT(cloneNFT).initialize(_controller, address(clone), _name, _symbol, _tokenURI, _seed);**
            // R [ISSUE][MEDIUM]: it is recommended to add gardenNFT as a valid systemcontroller to avoid malicious gardenNFT passed as params.
            Garden(clone).initialize{value: msg.value}(
                _reserveAsset,
    ```
    
    ```jsx
    contract GardenFactory is IGardenFactory {
        address private immutable garden;
        address private immutable gardenNFT;
    
        constructor(**//HERE AS PARAM**) {
            garden = address(new Garden());
            gardenNFT = address(new GardenNFT());
    				**// HERE**
        }
    ```
    
    ```jsx
    function initialize(
            **address _controller,**
            address _garden,
            string memory _name,
            string memory _symbol,
            string memory _gardenTokenURI,
            uint256 _seed
           //notion // R [ISSUE][MEDIUM] Check with Jota if there could be specific modifiers for initializing flow (avoiding malicious initializations) e.g. Official GardenFactory address as msg.sender 
        ) external override initializer {
            require(address(_controller) != address(0), 'Controller must exist');
            __ERC721_init(_name, _symbol);
            **// R [ISSUE][MEDIUM] Check the controller is the real controller or at least the Garden a controller Garden (isSystemContract), the problem is to do it while deploying
            controller = IBabController(_controller);**
            garden = IGarden(_garden);
            seed = _seed;
            gardenTokenURI = _gardenTokenURI;
        }
    ```
    
    ```jsx
    function initialize(
            address _strategist,
            **address _garden,
            address _controller,**
            uint256 _maxCapitalRequested,
            uint256 _stake,
            uint256 _strategyDuration,
            uint256 _expectedReturn,
            uint256 _minRebalanceCapital,
            address _strategyNft
        ) external override initializer {
            **// R [ISSUE][MEDIUM] It is recommended that controller param came from the factory as global variable set-up by its constructur taking into account that it is upgradeable (always the same)
            controller = IBabController(_controller);**
    
            **_require(controller.isSystemContract(_garden), Errors.NOT_A_GARDEN);**
    ```
    
    ```jsx
    function createStrategy(
            string memory _name,
            string memory _symbol,
            address _strategist,
            address _garden,
            **// R [ISSUE][MEDIUM] It is recommended that controller is a global variable not passed as param initialized in constructor. Then we avoid mistakes or malicious behavior trying to create new strategies by using different controllers
            address _controller,**
    ```
    
6. // R [ISSUE][MEDIUM] If the caller is a system contract, it is recommended to use isSystemContract as modifier to check whether or not the caller is a system contract. 
(Refs. GardenFactory.sol)
    
    <aside>
    💡 SOLVED, this fixes others as well (e.g. arbitrary callers).
    
    </aside>
    
    ```jsx
    function createGarden(
            address _reserveAsset,
            address _controller,
            address _creator,
            string memory _name,
            string memory _symbol,
            string memory _tokenURI,
            uint256 _seed,
            uint256[] calldata _gardenParams
            // R [ISSUE][MEDIUM] It is recommended to use controller (upgradeable) address for example initializing it in the constructor instead of using it as param each time, it help to avoid mistakes but also to avoid malicious potential behavior
            **// R [ISSUE][MEDIUM] If the caller is a system contract, it is recommended to use isSystemContract as modifier to check whether or not the caller is a system contract.**
        ) external payable override returns (address) {
            address payable clone = payable(Clones.clone(garden));
            address cloneNFT = Clones.clone(gardenNFT);
            GardenNFT(cloneNFT).initialize(_controller, address(clone), _name, _symbol, _tokenURI, _seed);
            // R [ISSUE][MEDIUM]: it is recommended to add gardenNFT as a valid systemcontroller to avoid malicious gardenNFT passed as params.
            Garden(clone).initialize{value: msg.value}(
                _reserveAsset,
    ```
    
7. // R [ISSUE][MEDIUM] onlyGarden should check if the Garden is the real Garden known by the controller at least after the creator/deployment (for normal users). The problem is that this is not possible as GardenNFT instance goes before Garden instance.
(Refs. GardenNFT.sol line 101)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    function grantGardenNFT(address _user) external override onlyGarden returns (uint256) {
            // R [ISSUE][MEDIUM] onlyGarden should check if the Garden is the real Garden known by the controller at least after the creator/deployment (for normal users)
            require(address(_user) != address(0), 'User must exist');
            return _createOrGetGardenNFT(_user);
        }
    ```
    
    ```jsx
    /* ============ Modifiers ============ */
    
        modifier onlyGarden {
            // R [ISSUE][MEDIUM] It is recommended to check if Garden is a system contract Garden to avoid malicious gardens
            require(msg.sender == address(garden), 'Only the garden can mint the NFT');
            _;
        }
    ```
    
8. // R [ISSUE][MEDIUM] It is recommended to use isSystemContract to really check that the garden is a real garden known by the controller and that the controller is the real one not a malicious one
(Refs. Strategy.sol lines 85, 95, 109, 138)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    modifier onlyGovernorOrGarden {
            **// R [ISSUE][MEDIUM] It is recommended to use isSystemContract to really check that the garden is a real garden known by the controller
            _require(msg.sender == address(garden) || msg.sender == controller.owner(), Errors.ONLY_PROTOCOL_OR_GARDEN);**
            _;
        }
    ```
    
    ```jsx
    modifier onlyContributor {
            **// R [ISSUE][MEDIUM] It is recommended to use isSystemContract to really check that the garden is a real garden known by the controller
            _require(IERC20(address(garden)).balanceOf(msg.sender) > 0, Errors.ONLY_CONTRIBUTOR);
            _;**
        }
    ```
    
    ```jsx
    modifier onlyGardenAndNotSet() {
            **// R [ISSUE][MEDIUM] It is recommended a isGarden (isSystemContract) check within the controller
            _require(msg.sender == address(garden) && !dataSet, Errors.ONLY_GARDEN_AND_DATA_NOT_SET);
            _;**
        }
    ```
    
    ```jsx
    modifier onlyActiveGarden() {
            **// R [ISSUE][MEDIUM] It is recommended a isGarden (isSystemContract) check within the controller
            _require(garden.active() == true, Errors.ONLY_ACTIVE_GARDEN);
            _;**
        }
    ```
    
9. // R [ISSUE][MEDIUM] Reserve asset might not be WETH so there is a need to assure compatibility
(Refs. Garden.sol lines 475)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    function payKeeper(address payable _keeper, uint256 _fee) external override {
            // R [ISSUE][HIGH] Anyone can execute paykeeper to pay the keeper at arbitrary times reducing garden balance without consent (e.g. DoS)
            _require(IBabController(controller).isValidKeeper(_keeper), Errors.ONLY_KEEPER);
            keeperDebt = keeperDebt.add(_fee);
            // Pay Keeper in WETH
            // R [ISSUE][INFORMATIVE] Pending TODO in the code under TOOD maybe not found after a quick TODO check.
            **// R [ISSUE][MEDIUM] Reserve asset might not be WETH so there is a need to assure compatibility**
            **// TOOD: Update principal
            // TOOD: Reserve asset may be not WETH
            if (keeperDebt > 0 && IERC20(reserveAsset).balanceOf(address(this)) >= keeperDebt) {
                IERC20(reserveAsset).safeTransfer(_keeper, keeperDebt);
                principal = principal.sub(keeperDebt);
                keeperDebt = 0;**
            }
        }
    ```
    
10. // R [ISSUE][MEDIUM] It supposes WETH is going to be at least one of the paid components
(Refs. PriceOracle.sol line 173)
    
    <aside>
    💡 SOLVED. Increased severity from INFORMATIVE to MEDIUM.
    
    </aside>
    
    ```jsx
    function _getPriceFromUniswapAnchoredView(address _assetOne, address _assetTwo)
            internal
            view
            returns (bool, uint256)
        {
            address assetToCheck = _assetOne;
            **// R [ISSUE][MEDIUM] It supposes WETH is going to be at least one of the paid components
            if (_assetOne == WETH) {
                assetToCheck = _assetTwo;**
            }
            if (
                assetToCheck == 0x6B175474E89094C44Da98b954EedeAC495271d0F || // dai
                assetToCheck == 0x1985365e9f78359a9B6AD760e32412f4a445E862 || // rep
                assetToCheck == 0xE41d2489571d322189246DaFA5ebDe1F4699F498 || // zrx
                assetToCheck == 0x0D8775F648430679A709E98d2b0Cb6250d2887EF || // bat
                assetToCheck == 0xdd974D5C2e2928deA5F71b9825b8b646686BD200 || // knc
                assetToCheck == 0x514910771AF9Ca656af840dff83E8264EcF986CA || // link
                assetToCheck == 0xc00e94Cb662C3520282E6f5717214004A7f26888 || // comp
                assetToCheck == 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 || // USDC
                assetToCheck == 0x1f9840a85d5aF5bf1D1762F925BDADdC4201F984 // uni
            ) {
                string memory symbol1 = _assetOne == WETH ? 'ETH' : ERC20(_assetOne).symbol();
                string memory symbol2 = _assetTwo == WETH ? 'ETH' : ERC20(_assetTwo).symbol();
                uint256 assetOnePrice = IUniswapAnchoredView(uniswapAnchoredView).price(symbol1);
                uint256 assetTwoPrice = IUniswapAnchoredView(uniswapAnchoredView).price(symbol2);
    
                if (assetOnePrice > 0 && assetTwoPrice > 0) {
                    return (true, assetOnePrice.preciseDiv(assetTwoPrice));
                }
            }
    
            return (false, 0);
        }
    ```
    
11. // R [ISSUE][MEDIUM] If reserve asset is not WETH but USDC (6 decimals) the fee should be using 6 decimals as well. Please check @Igor Yalovoy 
(Refs. Strategy.sol line 818)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    **function _transferStrategyPrincipal(uint256 _fee) internal {**
            // R [ISSUE][LOW] The chosen fee has a direct impact on the capital returned. As there are several implications around capital returned, expected return, etc. if the fee is too high, people might complaint 
            **//notion // R [ISSUE][MEDIUM] If reserve asset is not WETH but USDC (6 decimals) the fee should be using 6 decimals as well.
            capitalReturned = IERC20(garden.reserveAsset()).balanceOf(address(this)).sub(_fee);**
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
    
12. // R [ISSUE][MEDIUM] We need to confirm that reserveAssetDelta (reserveAsset) and burningAmount (garden ERC20) are using the same decimals and this operation makes sense
(Refs. Strategy.sol line 848)
    
    <aside>
    💡 SOLVED
    
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
                **// R [ISSUE][MEDIUM] We need to confirm that reserveAssetDelta (reserveAsset) and burningAmount (garden ERC20) are using the same decimals
                reserveAssetDelta.add(int256(burningAmount));**
            }
    ```
    
13. // R [ISSUE][MEDIUM] absoluteReturns is not being updated unless we do absoluteReturns = absoluteReturns.add(_returns). As it is a param not used for anything else yet, we mark it as medium but it might produce an impact on the brand if users see absoluteReturns= 0 along the time.
(Refs. Garden.sol line 457)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/312)
    
    </aside>
    
    ```jsx
    * @param _amount                        Amount of WETH to convert to ETH to set aside until the window ends
         * @param _rewards                       Amount of WETH to convert to ETH to set aside forever
         * @param _returns                       Profits or losses that the strategy received
         */
        function startWithdrawalWindow(
            uint256 _amount,
            uint256 _rewards,
            int256 _returns,
            address _strategy
        ) external override {
            // R [ISSUE][HIGH] If we call it more than once (e.g. 1st from a strategy and 2nd or even more by controller) we will create a mess duplicating reserveAssetRewards and withdrawaling twice
            _require(
                (strategyMapping[msg.sender] && address(IStrategy(msg.sender).garden()) == address(this)) ||
                    msg.sender == controller,
                Errors.ONLY_STRATEGY_OR_CONTROLLER
            );
            // Updates reserve asset
            principal = principal.toInt256().add(_returns).toUint256();
            if (withdrawalsOpenUntil > block.timestamp) {
                withdrawalsOpenUntil = block.timestamp.add(
                    withdrawalWindowAfterStrategyCompletes.sub(withdrawalsOpenUntil.sub(block.timestamp))
                );
            } else {
                withdrawalsOpenUntil = block.timestamp.add(withdrawalWindowAfterStrategyCompletes);
            }
            reserveAssetRewardsSetAside = reserveAssetRewardsSetAside.add(_rewards);
            reserveAssetPrincipalWindow = reserveAssetPrincipalWindow.add(_amount);
            // Both are converted to weth
            IWETH(WETH).withdraw(_amount.add(_rewards));
    
            // Mark strategy as finalized
            **// R [ISSUE][MEDIUM] absoluteReturns is not being updated unless we do absoluteReturns = absoluteReturns.add(_returns)
            absoluteReturns.add(_returns);**
            strategies = strategies.remove(_strategy);
            finalizedStrategies.push(_strategy);
            strategyMapping[_strategy] = false;
        }
    ```
    
14. // R [ISSUE][MEDIUM] A user can increase allowance above its unlockedBalance doing it by smaller chunks
(Refs. TimeLockedToken.sol line 390)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/329)
    
    </aside>
    
    ```jsx
    * function increaseAllowance(address spender, uint256 addedValue) public override nonReentrant returns (bool) {
            **// R [ISSUE][MEDIUM] A user can increase above its unlockedBalance doing it by smaller chunks
            require(
                unlockedBalance(msg.sender) >= addedValue,
                'TimeLockedToken::increaseAllowance:Not enough unlocked tokens'
            );**
            require(spender != address(0), 'TimeLockedToken::increaseAllowance:Spender cannot be zero address');
            require(spender != msg.sender, 'TimeLockedToken::increaseAllowance:Spender cannot be the msg.sender');
            _approve(msg.sender, spender, allowance(msg.sender, spender).add(addedValue));
            return true;
        }
    ```
    
15. // R [ISSUE][MEDIUM] Vote delegation by signature was not adding the prefix by signed messages so erecover from ECDSA was not recovering the right address of delegator who signed the message.
(Refs. VoteToken.sol line 123)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/347/commits/57bbfa2dfaa243ba6c6594ac626dca8065b284bb)
    
    </aside>
    
    <aside>
    💡 Comments: Adding the prefix and hashing the message solves the problem for metamask signatures but we leave the option to use prefix or not while we continue making tests as there are others not using the hash prefix:
    COMP: [https://github.com/compound-finance/compound-protocol/blob/master/contracts/Governance/Comp.sol](https://github.com/compound-finance/compound-protocol/blob/master/contracts/Governance/Comp.sol) → implementation using method: 'eth_signTypedData_v4', here: [https://github.com/compound-developers/compound-governance-examples/blob/master/signature-examples/delegate_by_signature.html](https://github.com/compound-developers/compound-governance-examples/blob/master/signature-examples/delegate_by_signature.html) 
    TRUSTTOKEN: [https://github.com/trusttoken/smart-contracts/blob/master/contracts/governance/VoteToken.sol](https://github.com/trusttoken/smart-contracts/blob/master/contracts/governance/VoteToken.sol) 
    SUSHISWAP: [https://github.com/sushiswap/sushiswap/blob/master/contracts/SushiToken.sol](https://github.com/sushiswap/sushiswap/blob/master/contracts/SushiToken.sol)
    
    </aside>
    
    ```jsx
    * function delegateBySig(
            address delegatee,
            uint256 nonce,
            uint256 expiry,
            uint8 v,
            bytes32 r,
            bytes32 s
        ) external override {
            bytes32 domainSeparator =
                keccak256(abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name())), getChainId(), address(this)));
            bytes32 structHash = keccak256(abi.encode(DELEGATION_TYPEHASH, delegatee, nonce, expiry));
            **bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));
            address signatory = ecrecover(digest, v, r, s);**
            require(signatory != address(0), 'VoteToken::delegateBySig: invalid signature');
            require(nonce == nonces[signatory] + 1, 'VoteToken::delegateBySig: invalid nonce');
            nonces[signatory]++;
            require(block.timestamp <= expiry, 'VoteToken::delegateBySig: signature expired');
            return _delegate(signatory, delegatee);
        }
    ```
    
16. // R [ISSUE][MEDIUM] Vote delegation by signature was stucked as nonce needs to match with the one used in signature before it is increased by ++. StructHash needs to use the same nonce used in the signature process but it was expecting +1 so it might never work.
(Refs. VoteToken.sol line 125)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/347/commits/57bbfa2dfaa243ba6c6594ac626dca8065b284bb)
    
    </aside>
    
    ```jsx
    * function delegateBySig(
            address delegatee,
            uint256 nonce,
            uint256 expiry,
            uint8 v,
            bytes32 r,
            bytes32 s
        ) external override {
            bytes32 domainSeparator =
                keccak256(abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name())), getChainId(), address(this)));
            **bytes32 structHash = keccak256(abi.encode(DELEGATION_TYPEHASH, delegatee, nonce, expiry));**
            bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));
            address signatory = ecrecover(digest, v, r, s);
            require(signatory != address(0), 'VoteToken::delegateBySig: invalid signature');
            **require(nonce == nonces[signatory] + 1, 'VoteToken::delegateBySig: invalid nonce');**
            nonces[signatory]++;
            require(block.timestamp <= expiry, 'VoteToken::delegateBySig: signature expired');
            return _delegate(signatory, delegatee);
        }
    ```
    
17. // R [ISSUE][MEDIUM] A malicious smartcontract could potentially impersonate a strategy to add protocolPrincipal as the modifier only checked that its garden variable is a official garden but it a threat actor can set up that variable in a malicious smartcontract. A double (cross) check is needed → .garden() is a official garden and that official garden knows about that strategy (protocol strategy).
(Refs. RewardsDistributor.sol line 73)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/351/commits/65d4ea81e5596d7915c39e0e7fb62655d374f42b)
    
    </aside>
    
    ```jsx
    * modifier onlyStrategy {
            **_require(controller.isSystemContract(address(IStrategy(msg.sender).garden())), Errors.ONLY_STRATEGY);**
            _;
        }
    ```
    
18. // R [ISSUE][MEDIUM] As BABL Token deployment is being delayed, RewardsDistributor needs a setter to be able to support an address change.
(Refs. RewardsDistributor.sol line 217 and 262)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/368)
    
    </aside>
    
    ```jsx
    * // BABL Token contract
        TimeLockedToken public babltoken;
    ```
    
    ```jsx
    * function initialize(TimeLockedToken _bablToken, IBabController _controller) public {
            OwnableUpgradeable.__Ownable_init();
            _require(address(_bablToken) != address(0), Errors.ADDRESS_IS_ZERO);
            _require(address(_controller) != address(0), Errors.ADDRESS_IS_ZERO);
            **babltoken = _bablToken;**
            controller = _controller;
    
            (BABL_STRATEGIST_SHARE, BABL_STEWARD_SHARE, BABL_LP_SHARE, CREATOR_BONUS) = controller.getBABLSharing();
            (PROFIT_STRATEGIST_SHARE, PROFIT_STEWARD_SHARE, PROFIT_LP_SHARE) = controller.getProfitSharing();
            PROFIT_PROTOCOL_FEE = controller.protocolPerformanceFee();
    
            status = NOT_ENTERED;
        }
    ```
    
19. // R [ISSUE][MEDIUM] Deployment is failing due to a not needed modifier _onlyStrategy in Lend getNAV external "view" function.
(Refs. LendIntegration.sol line 166)
    
    <aside>
    💡 SOLVED (https://github.com/babylon-finance/protocol/pull/368)
    
    </aside>
    
    ```jsx
    * /**
         * Gets the NAV of the lend op in the reserve asset
         *
         * @param _assetToken         Asset lent
         * @param _garden             Garden the strategy belongs to
         * @param _integration        Status of the asset amount
         * @return _nav           NAV of the strategy
         */
        function getNAV(
            address _assetToken,
            IGarden _garden,
            address _integration
        ) external view override **onlyStrategy** returns (uint256, bool) {
            if (!IStrategy(msg.sender).isStrategyActive()) {
                return (0, true);
            }
    ```