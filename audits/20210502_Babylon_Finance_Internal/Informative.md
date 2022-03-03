# Informative

1. // R [ISSUE][INFORMATIVE] - Just to check that 1 January 2022 is the correct date. The unixtimestamp is not ok (javascript adds x1000 → remove)
(Refs. BabController.sol line 264)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    function enableGardenTokensTransfers() external override onlyOwner {
            **// R [ISSUE][INFORMATIVE] - Just to check that 1 January 2022 is the correct date
            require(block.timestamp > 1641024000000,** 'Transfers cannot be enabled yet');
            gardenTokensTransfersEnabled = true;
        }
    ```
    
2. // R [ISSUE][INFORMATIVE] update param _quoteAsset
(Refs. BabController.sol line 264)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    /**
         * Gets the valuation of a Garden using data from the price oracle.
         * Adds all the active strategies plus the reserve asset and ETH.
         * Note: this works for external
         * positions and negative (debt) positions.
         *
         * Note: There is a risk that the valuation is off if airdrops aren't retrieved
         *
         * @param _garden          Garden instance to get valuation
         **// R [ISSUE][INFORMATIVE] update param _quoteAsset**
         *
         * @return                 Token valuation in terms of quote asset in precise units 1e18
         */
        function calculateGardenValuation(address _garden, **address _quoteAsset**) external view returns (uint256) {
            IPriceOracle priceOracle = IPriceOracle(IBabController(controller).priceOracle());
            address reserveAsset = IGarden(_garden).reserveAsset();
    ```
    
3. // R [ISSUE][INFORMATIVE] Check that permissions are <= 3
(Refs. IshtarGate.sol line 264)
    
    <aside>
    💡 SOLVED ([https://github.com/babylon-finance/protocol/pull/315](https://github.com/babylon-finance/protocol/pull/315))
    
    </aside>
    
    ```jsx
    function grantGardenAccessBatch(
            address _garden,
            address[] calldata _users,
            uint8[] calldata _perms
        ) external override onlyGardenCreator(_garden) returns (bool) {
            require(_users.length == _perms.length, 'Permissions and users must match');
            for (uint8 i = 0; i < _users.length; i++) {
                require(address(_users[i]) != address(0), 'User must exist');
                **// R [ISSUE][INFORMATIVE] Check that permissions are <= 3
                _setIndividualGardenAccess(_users[i], _garden, _perms[i]);
            }**
            return true;
        }
    ```
    
4. // R [ISSUE][INFORMATIVE] Creator permissions here can also be removing access (check description above)
(Refs. IshtarGate.sol line 184)
    
    <aside>
    💡 ACCEPTED BY DESIGN
    
    </aside>
    
    ```jsx
    /**
         *** Awards the ishtar gate to a list of users with permissions to create gardens**
         *
         * @param _users              Addresses of the users
         * @param _perms              Lists of booleans
         */
        function grantCreatorsInBatch(address[] calldata _users, bool[] calldata _perms)
            external
            override
            onlyOwner
            returns (bool)
        {
            require(_users.length == _perms.length, 'Permissions and users must match');
            for (uint8 i = 0; i < _users.length; i++) {
                // R [ISSUE][INFORMATIVE] Creator permissions here can also be removing access (check description above)
                _setCreatorPermissions(_users[i], _perms[i]);
            }
            return true;
        }
    ```
    
5. // R [ISSUE][INFORMATIVE] Just to follow standard increment like the rest of the code it is preferred i++ instead of i += 1
(Refs. PriceOracle.sol line 158)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    /**
         * Calls the update function in every adapter.
         * e.g Uniswap TWAP
         * @param _assetOne       First Asset of the pair
         * @param _assetTwo       Second Asset of the pair
         */
        function updateAdapters(address _assetOne, address _assetTwo) external override {
            // R [ISSUE][INFORMATIVE] Just to follow standard increment like the rest of the code it is preferred i++ instead of i += 1
            for (uint256 i = 0; i < adapters.length; **i += 1)** {
                //notion // R [ISSUE][CHECK] Check potential result if a malicious update of a non existent pair (or not WETH related) is executed 
                IOracleAdapter(adapters[i]).update(_assetOne, _assetTwo);
            }
        }
    ```
    
6. // R [ISSUE][INFORMATIVE] Adjust this calculation stake > 0
(Refs. Strategy.sol line 271)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    * Initializes the strategy for a garden
         *
         * @param _strategist                    Address of the strategist
         * @param _garden                        Address of the garden
         * @param _controller                    Address of the controller
         * @param _maxCapitalRequested           Max Capital requested denominated in the reserve asset (0 to be unlimited)
         * @param _stake                         Stake with garden participations absolute amounts 1e18
         * @param _strategyDuration              Strategy duration in seconds
         * @param _expectedReturn                Expected return
         * @param _minRebalanceCapital           Min capital that makes executing the strategy worth it
         * @param _strategyNft                   Address of the strategy nft
         */
        function initialize(
            address _strategist,
            address _garden,
            address _controller,
            uint256 _maxCapitalRequested,
            uint256 _stake,
            uint256 _strategyDuration,
            uint256 _expectedReturn,
            uint256 _minRebalanceCapital,
            address _strategyNft
        ) external override initializer {
            //notion // R [ISSUE][MEDIUM] It is recommended that controller param came from the factory as global variable set-up by its constructur taking into account that it is upgradeable and inmutable (always the same)
            controller = IBabController(_controller);
    
            _require(controller.isSystemContract(_garden), Errors.NOT_A_GARDEN);
            garden = IGarden(_garden);
            uint256 strategistUnlockedBalance =
                IERC20(address(garden)).balanceOf(_strategist).sub(garden.getLockedBalance(_strategist));
            // R [ISSUE][INFORMATIVE] >0 is less restrictive than the following one, maybe we can save gas w/o this check (>0). Take care about garden tokens vs. stake reserve asset differences (if any)
            _require(IERC20(address(garden)).balanceOf(_strategist) > 0, Errors.STRATEGIST_TOKENS_TOO_LOW);
            _require(strategistUnlockedBalance >= _stake, Errors.TOKENS_STAKED);
            // R [ISSUE][INFORMATIVE] Adjust this calculation, not sure the objective of the adjusting (check with Ramon)
            **// TODO: adjust this calc
            _require(_stake > 0, Errors.STAKE_HAS_TO_AT_LEAST_ONE);**
            _require(
                _strategyDuration >= garden.minStrategyDuration() && _strategyDuration <= garden.maxStrategyDuration(),
                Errors.DURATION_MUST_BE_IN_RANGE
            );
    ```
    
7. // R [ISSUE][INFORMATIVE] It is recommended to check that strategyNFT is a valid controller strategyNFT beyond != address(0)
(Refs. Strategy.sol line 280)
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    **_require(_strategyNft != address(0), Errors.NOT_STRATEGY_NFT);
            // R [ISSUE][INFORMATIVE] It is recommended to check that strategyNFT is a valid controller strategyNFT beyond != address(0)
            strategyNft = _strategyNft;**
    ```
    
8. // R [ISSUE][INFORMATIVE] Shouldn't we check if garden has any balance still to drain it to treasury, users or whatever before removing and disabling? In that case what are we going to do with it? (Refs. BabController.sol lines 223)
    
    <aside>
    💡 DOWNSIZED TO [INFORMATIVE] from MEDIUM
    
    </aside>
    
    <aside>
    💡 SOLVED
    
    </aside>
    
    ```jsx
    function removeGarden(address _garden) external override onlyOwner {
            **// R [ISSUE][MEDIUM] Shouldn't we check if garden has any balance still to drain it to treasury, users or whatever? In that case what are we going to do with it?**
            require(isGarden[_garden], 'Garden does not exist');
            require(!IGarden(_garden).active(), 'The garden needs to be disabled.');
            **gardens = gardens.remove(_garden);**
    
            delete isGarden[_garden];
    
            emit GardenRemoved(_garden);
        }
    ```
    
    ```jsx
    function disableGarden(address _garden) external override onlyOwner {
            require(isGarden[_garden], 'Garden does not exist');
            IGarden garden = IGarden(_garden);
            require(garden.active(), 'The garden needs to be active.');
            **// R [ISSUE][MEDIUM] Shouldn't we check if garden has any balance still to drain it to treasury, users or whatever? In that case what are we going to do with it?
            garden.setActive(false);**
        }
    ```