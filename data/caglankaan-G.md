### Unnecessary Variable Initialization
In programming, it's a common practice to declare and initialize variables that will be used later in the code. However, if after declaration, the variable is not used anywhere else in the code, this leads to unnecessary variable initialization. In the context of Solidity and smart contracts, where gas efficiency is paramount, such redundant initializations not only consume extra gas but also clutter the codebase, making it less readable and harder to maintain.

Variables that are declared but never used represent dead code. They take up space in the bytecode, result in wasted gas when the contract is deployed, and can be confusing for anyone reading or auditing the code.


```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

280:            bytes32 _cdpId = borrowerOperations.openCdpFor(	// @audit-issue
```
[280](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L280-L280), 


#### Recommendation

To avoid unnecessary variable initialization in Solidity code, regularly review and remove unused variables, use meaningful variable names, and avoid overly defensive coding. Prioritize gas efficiency and rigorous testing to maintain clean and efficient code.

### Same cast is done multiple times
It's cheaper to do it once, and store the result to a variable.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

56:        stETH.approve(address(borrowerOperations), type(uint256).max);	// @audit-issue: Same casting also done in line(s): `[55]`.

104:        op.tokenToTransferIn = address(stETH);	// @audit-issue: Same casting also done in line(s): `[125, 118]`.

117:                address(ebtcToken), 	// @audit-issue: Same casting also done in line(s): `[126]`.

178:        op.swapsAfter = _getSwapOperations(address(ebtcToken), address(stETH), _tradeData);	// @audit-issue: Same casting also done in line(s): `[173]`.
```
[56](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L56-L56), [104](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L104-L104), [117](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L117-L117), [178](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L178-L178), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

80:            uint256 _wstETHVal = IWstETH(address(wstEth)).wrap(_stEthVal);	// @audit-issue: Same casting also done in line(s): `[85]`.
```
[80](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L80-L80), 


#### Recommendation

Identify instances in your Solidity contracts where the same value is cast to another type multiple times. Refactor these operations by performing the cast once and storing the result in a local variable. Use this variable for all subsequent operations requiring the cast value.

### `address(this)` should be cached
Cacheing saves gas when compared to repeating the calculation at each point it is used in the contract.

```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

300:        uint256 _zapStEthBalanceAfter = stEth.balanceOf(address(this));	// @audit-issue: `adress(this)` also used on line(s): [294]
```
[300](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L300-L300), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

39:        uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;	// @audit-issue: `adress(this)` also used on line(s): [36]

44:        uint256 _wETHBalBefore = wrappedEth.balanceOf(address(this));	// @audit-issue: `adress(this)` also used on line(s): [48, 45, 50, 46]

67:        uint256 _stETHReiceived = stEth.balanceOf(address(this)) - _stETHBalBefore;	// @audit-issue: `adress(this)` also used on line(s): [65, 61]

111:        uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;	// @audit-issue: `adress(this)` also used on line(s): [110, 109]
```
[39](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L39-L39), [44](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L44-L44), [67](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L67-L67), [111](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L111-L111), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

187:                IERC3156FlashBorrower(address(this)),	// @audit-issue: `adress(this)` also used on line(s): [179, 194]

254:        uint256 collateralBal = stETH.sharesOf(address(this));	// @audit-issue: `adress(this)` also used on line(s): [253]
```
[187](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L187-L187), [254](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L254-L254), 


#### Recommendation

To enhance gas efficiency, cache the contract's address by storing `address(this)` in a state variable at the point of contract deployment or initialization. Use this cached address throughout the contract instead of repeatedly calling `address(this)`. This practice reduces the gas cost associated with multiple computations of the contract's address, leading to more efficient contract execution, especially in scenarios with frequent usage of the contract's address.

### Revert String Size Optimization
Shortening the revert strings to fit within 32 bytes will decrease deployment time and decrease runtime Gas when the revert condition is met.

Revert strings that are longer than 32 bytes require at least one additional `mstore`, along with additional overhead to calculate memory offset, etc.



```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

113:        require(	// @audit-issue: String length is `60`
114:            msg.sender == address(wrappedEth),
115:            "EbtcLeverageZapRouter: only allow Wrapped ETH to send Ether!"
116:        );

282:        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for close!");	// @audit-issue: String length is `43`

315:        require(	// @audit-issue: String length is `68`
316:            _stEthBalanceIncrease > 0 || _stEthBalanceDecrease > 0 || _debtChange > 0,
317:            "BorrowerOperations: There must be either a collateral or debt change"
318:        );

325:        require(	// @audit-issue: String length is `71`
326:            _stEthMarginIncrease == 0 || _stEthMarginDecrease == 0,
327:            "EbtcLeverageZapRouter: Cannot add and withdraw margin in same operation"
328:        );

410:        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for adjust!");	// @audit-issue: String length is `44`
```
[113](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L113-L116), [282](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L282-L282), [315](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L315-L318), [325](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L325-L328), [410](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L410-L410), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

55:        require(msg.value == _initialETH, "EbtcZapRouter: Incorrect ETH amount");	// @audit-issue: String length is `35`

60:        require(	// @audit-issue: String length is `39`
61:            wstEth.transferFrom(msg.sender, address(this), _initialWstETH),
62:            "EbtcZapRouter: transfer wstETH failure!"
63:        );

136:        require(	// @audit-issue: String length is `66`
137:            _change == 0 || _change >= MIN_CHANGE,
138:            "ZapRouterBase: Debt or collateral change must be zero or above min"
139:        );

143:        require(	// @audit-issue: String length is `66`
144:            _stEthBalance >= MIN_NET_STETH_BALANCE,
145:            "ZapRouterBase: Cdp's net stEth balance must not fall below minimum"
146:        );
```
[55](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L55-L55), [60](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L60-L63), [136](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L136-L139), [143](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L143-L146), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

212:            require(	// @audit-issue: String length is `45`
213:                cdpInfo.status == checkParams.expectedStatus,
214:                "!LeverageMacroReference: openCDP status check"
215:            );

224:            require(	// @audit-issue: String length is `47`
225:                cdpInfo.status == checkParams.expectedStatus,
226:                "!LeverageMacroReference: adjustCDP status check"
227:            );

234:            require(	// @audit-issue: String length is `46`
235:                cdpInfo.status == checkParams.expectedStatus,
236:                "!LeverageMacroReference: closeCDP status check"
237:            );

282:            require(check.value >= valueToCheck, "!LeverageMacroReference: gte post check");	// @audit-issue: String length is `39`

284:            require(check.value <= valueToCheck, "!LeverageMacroReference: let post check");	// @audit-issue: String length is `39`

286:            require(check.value == valueToCheck, "!LeverageMacroReference: equal post check");	// @audit-issue: String length is `41`

402:        require(initiator == address(this), "LeverageMacroReference: wrong initiator for flashloan");	// @audit-issue: String length is `53`

406:            require(	// @audit-issue: String length is `55`
407:                msg.sender == address(borrowerOperations),
408:                "LeverageMacroReference: wrong lender for eBTC flashloan"
409:            );

412:            require(	// @audit-issue: String length is `56`
413:                msg.sender == address(activePool),
414:                "LeverageMacroReference: wrong lender for stETH flashloan"
415:            );

490:                require(	// @audit-issue: String length is `43`
491:                    IERC20(swapChecks[i].tokenToCheck).balanceOf(address(this)) >
492:                        swapChecks[i].expectedMinOut,
493:                    "LeverageMacroReference: swap check failure!"
494:                );
```
[212](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L212-L215), [224](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L224-L227), [234](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L234-L237), [282](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L282-L282), [284](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L284-L284), [286](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L286-L286), [402](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L402-L402), [406](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L406-L409), [412](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L412-L415), [490](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L490-L494), 


#### Recommendation


To optimize Gas usage in your Solidity contract, it is recommended to keep revert strings as short as possible and to ensure that they fit within 32 bytes. It is possible to use abbreviations or simplified error messages to keep the string length short. Doing so can reduce the amount of Gas used during deployment and runtime when the revert condition is met.


### Custom Errors in Solidity for Gas Efficiency
Starting from Solidity version 0.8.4, the language introduced a feature known as "custom errors". These custom errors provide a way for developers to define more descriptive and semantically meaningful error conditions without relying on string messages. Prior to this version, developers often used the `require` statement with string error messages to handle specific conditions or validations. However, every unique string used as a revert reason consumes gas, making transactions more expensive.

Custom errors, on the other hand, are identified by their name and the types of their parameters only, and they do not have the overhead of string storage. This means that, when using custom errors instead of `require` statements with string messages, the gas consumption can be significantly reduced, leading to more gas-efficient contracts.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

46:            require(params.zapFeeReceiver != address(0));

73:        revert("disabled");	// @audit-issue
```
[45](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L46-L46), [73](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L73-L73), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

113:        require(	// @audit-issue
114:            msg.sender == address(wrappedEth),
115:            "EbtcLeverageZapRouter: only allow Wrapped ETH to send Ether!"
116:        );

282:        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for close!");	// @audit-issue

315:        require(	// @audit-issue
316:            _stEthBalanceIncrease > 0 || _stEthBalanceDecrease > 0 || _debtChange > 0,
317:            "BorrowerOperations: There must be either a collateral or debt change"
318:        );

325:        require(	// @audit-issue
326:            _stEthMarginIncrease == 0 || _stEthMarginDecrease == 0,
327:            "EbtcLeverageZapRouter: Cannot add and withdraw margin in same operation"
328:        );

410:        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for adjust!");	// @audit-issue
```
[113](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L113-L116), [282](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L282-L282), [315](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L315-L318), [325](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L325-L328), [410](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L410-L410), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

55:        require(msg.value == _initialETH, "EbtcZapRouter: Incorrect ETH amount");	// @audit-issue

60:        require(	// @audit-issue
61:            wstEth.transferFrom(msg.sender, address(this), _initialWstETH),
62:            "EbtcZapRouter: transfer wstETH failure!"
63:        );

136:        require(	// @audit-issue
137:            _change == 0 || _change >= MIN_CHANGE,
138:            "ZapRouterBase: Debt or collateral change must be zero or above min"
139:        );

143:        require(	// @audit-issue
144:            _stEthBalance >= MIN_NET_STETH_BALANCE,
145:            "ZapRouterBase: Cdp's net stEth balance must not fall below minimum"
146:        );
```
[55](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L55-L55), [60](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L60-L63), [136](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L136-L139), [143](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L143-L146), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

40:        revert("Must be overridden");	// @audit-issue

45:        require(owner() == msg.sender, "Must be owner");	// @audit-issue

212:            require(
213:                cdpInfo.status == checkParams.expectedStatus,
214:                "!LeverageMacroReference: openCDP status check"
215:            );

224:            require(
225:                cdpInfo.status == checkParams.expectedStatus,
226:                "!LeverageMacroReference: adjustCDP status check"
227:            );

234:            require(
235:                cdpInfo.status == checkParams.expectedStatus,
236:                "!LeverageMacroReference: closeCDP status check"
237:            );

282:            require(check.value >= valueToCheck, "!LeverageMacroReference: gte post check");

284:            require(check.value <= valueToCheck, "!LeverageMacroReference: let post check");

286:            require(check.value == valueToCheck, "!LeverageMacroReference: equal post check");

288:            revert("Operator not found");

402:        require(initiator == address(this), "LeverageMacroReference: wrong initiator for flashloan");	// @audit-issue

406:            require(
407:                msg.sender == address(borrowerOperations),
408:                "LeverageMacroReference: wrong lender for eBTC flashloan"
409:            );

412:            require(
413:                msg.sender == address(activePool),
414:                "LeverageMacroReference: wrong lender for stETH flashloan"
415:            );

471:        require(success, "Call has failed");	// @audit-issue

490:                require(
491:                    IERC20(swapChecks[i].tokenToCheck).balanceOf(address(this)) >
492:                        swapChecks[i].expectedMinOut,
493:                    "LeverageMacroReference: swap check failure!"
494:                );

502:        require(addy != address(borrowerOperations));	// @audit-issue

503:        require(addy != address(sortedCdps));	// @audit-issue

504:        require(addy != address(activePool));	// @audit-issue

505:        require(addy != address(cdpManager));	// @audit-issue

506:        require(addy != address(this)); // If it could call this it could fake the forwarded caller	// @audit-issue
```
[40](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L40-L40), [45](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L45-L45), [207](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L212-L215), [219](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L224-L227), [231](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L234-L237), [278](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L282-L282), [278](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L284-L284), [278](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L286-L286), [278](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L288-L288), [402](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L402-L402), [405](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L406-L409), [405](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L412-L415), [471](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L471-L471), [487](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L490-L494), [502](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L502-L502), [503](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L503-L503), [504](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L504-L504), [505](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L505-L505), [506](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L506-L506), 


#### Recommendation


It is recommended to use custom errors instead of revert strings to reduce gas costs, especially during contract deployment. Custom errors can be defined using the error keyword and can include dynamic information.

### Avoid repeating computations
In Solidity development, repeating the same computations within a contract can lead to unnecessary gas consumption and reduce the contract's efficiency. This is particularly relevant in functions that are called frequently or involve complex calculations. Repeating computations not only wastes computational resources but also increases the cost of executing transactions. By identifying and eliminating redundant calculations, developers can optimize contract performance, reduce gas costs, and improve overall execution speed.

```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

222:        if (_positionManagerPermit.length > 0) {	// @audit-issue: Same binary operation statement in line(s) between: ['245:245']

289:        if (_positionManagerPermit.length > 0) {	// @audit-issue: Same binary operation statement in line(s) between: ['303:303']

418:        if (_positionManagerPermit.length > 0) {	// @audit-issue: Same binary operation statement in line(s) between: ['455:455']

424:        if (!_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue: Same binary operation statement in line(s) between: ['429:429']
```
[222](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L222-L222), [289](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L289-L289), [418](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L418-L418), [424](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L424-L424), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

132:            postCheckType != PostOperationCheck.none	// @audit-issue: Same binary operation statement in line(s) between: ['141:141']

213:                cdpInfo.status == checkParams.expectedStatus,	// @audit-issue: Same binary operation statement in line(s) between: ['225:225', '235:235']
```
[132](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L132-L132), [213](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L213-L213), 


#### Recommendation

Review your Solidity contracts to identify any computations that are performed multiple times with the same inputs. Cache the results of these computations in local variables and reuse them within the function or across function calls if the state remains unchanged.

### Unused State Variables
There are state variables which are declared but not used in any function. This might increase the contract's gas usage unnecessarily.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

18:    uint256 internal constant PRECISION = 1e18;	// @audit-issue
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L18-L18), 


#### Recommendation

To optimize your contract's gas usage, remove any state variables that are declared but not used in any function. Unnecessary state variables can increase gas costs and should be eliminated during code review.

### Divisions can be unchecked to save gas
The expression type(int).min/(-1) is the only case where division causes an overflow. Therefore, uncheck can be used to save gas in scenarios where it is certain that such an overflow will not occur.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

268:                    value:  _totalCollateral * _collValidationBuffer / BPS,	// @audit-issue

288:            IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData.eBTCToMint * zapFeeBPS / BPS);	// @audit-issue

309:                IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData._EBTCChange * zapFeeBPS / BPS);	// @audit-issue
```
[268](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L268-L268), [288](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L288-L288), [309](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L309-L309), 


#### Recommendation

Utilize 'unchecked' blocks in Solidity for divisions where overflow is impossible, such as when 'type(int).min/(-1)' is not a concern. This can save gas by bypassing overflow checks in these specific cases.

### Stack variable is only used once
If the variable is only accessed once, it's cheaper to use the assigned value directly that one time, and save the 3 gas the extra stack assignment would spend

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

141:        uint256 newDebt = _cdp._isDebtIncrease ? debt + _cdp._EBTCChange : debt - _cdp._EBTCChange;	// @audit-issue: newDebt used only on line: 153

142:        uint256 newColl = _cdp._stEthBalanceIncrease > 0 ? 	// @audit-issue: newColl used only on line: 154
143:            coll + stEth.getSharesByPooledEth(_cdp._stEthBalanceIncrease) : 
144:            coll - stEth.getSharesByPooledEth(_cdp._stEthBalanceDecrease);
```
[141](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L141-L141), [142](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L142-L144), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

223:            PositionManagerPermit memory approval = abi.decode(_positionManagerPermit, (PositionManagerPermit));	// @audit-issue: approval used only on line: 224

284:        uint256 debt = ICdpManager(address(cdpManager)).getSyncedCdpDebt(_cdpId);	// @audit-issue: debt used only on line: 297

290:            PositionManagerPermit memory approval = abi.decode(_positionManagerPermit, (PositionManagerPermit));	// @audit-issue: approval used only on line: 291

294:        uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));	// @audit-issue: _zapStEthBalanceBefore used only on line: 301

300:        uint256 _zapStEthBalanceAfter = stEth.balanceOf(address(this));	// @audit-issue: _zapStEthBalanceAfter used only on line: 301

301:        uint256 _stETHDiff = _zapStEthBalanceAfter - _zapStEthBalanceBefore;	// @audit-issue: _stETHDiff used only on line: 307

342:        uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));	// @audit-issue: _zapStEthBalanceBefore used only on line: 346

360:        uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));	// @audit-issue: _zapStEthBalanceBefore used only on line: 364

378:        uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));	// @audit-issue: _zapStEthBalanceBefore used only on line: 382

396:        uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));	// @audit-issue: _zapStEthBalanceBefore used only on line: 400

416:        (uint256 debt, uint256 coll) = ICdpManager(address(cdpManager)).getSyncedDebtAndCollShares(_cdpId);	// @audit-issue: debt used only on line: 449

416:        (uint256 debt, uint256 coll) = ICdpManager(address(cdpManager)).getSyncedDebtAndCollShares(_cdpId);	// @audit-issue: coll used only on line: 450

419:            PositionManagerPermit memory approval = abi.decode(_positionManagerPermit, (PositionManagerPermit));	// @audit-issue: approval used only on line: 420
```
[223](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L223-L223), [284](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L284-L284), [290](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L290-L290), [294](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L294-L294), [300](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L300-L300), [301](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L301-L301), [342](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L342-L342), [360](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L360-L360), [378](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L378-L378), [396](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L396-L396), [416](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L416-L416), [416](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L416-L416), [419](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L419-L419), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

36:        uint256 _balBefore = stEth.balanceOf(address(this));	// @audit-issue: _balBefore used only on line: 39

39:        uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;	// @audit-issue: _deposit used only on line: 40

44:        uint256 _wETHBalBefore = wrappedEth.balanceOf(address(this));	// @audit-issue: _wETHBalBefore used only on line: 46

46:        uint256 _wETHReiceived = wrappedEth.balanceOf(address(this)) - _wETHBalBefore;	// @audit-issue: _wETHReiceived used only on line: 49

48:        uint256 _rawETHBalBefore = address(this).balance;	// @audit-issue: _rawETHBalBefore used only on line: 50

50:        uint256 _rawETHConverted = address(this).balance - _rawETHBalBefore;	// @audit-issue: _rawETHConverted used only on line: 51

65:        uint256 _stETHBalBefore = stEth.balanceOf(address(this));	// @audit-issue: _stETHBalBefore used only on line: 67

67:        uint256 _stETHReiceived = stEth.balanceOf(address(this)) - _stETHBalBefore;	// @audit-issue: _stETHReiceived used only on line: 69

109:        uint256 _balBefore = stEth.balanceOf(address(this));	// @audit-issue: _balBefore used only on line: 111

111:        uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;	// @audit-issue: _deposit used only on line: 112

150:        uint256 _tmp = uint256(cdpId) >> 96;	// @audit-issue: _tmp used only on line: 151
```
[36](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L36-L36), [39](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L39-L39), [44](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L44-L44), [46](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L46-L46), [48](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L48-L48), [50](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L50-L50), [65](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L65-L65), [67](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L67-L67), [109](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L109-L109), [111](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L111-L111), [150](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L150-L150), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

143:            OpenCdpForOperation memory flData = abi.decode(	// @audit-issue: flData used only on line: 149
144:                operation.OperationData,
145:                (OpenCdpForOperation)
146:            );

232:            ICdpManagerData.Cdp memory cdpInfo = cdpManager.Cdps(checkParams.cdpId);	// @audit-issue: cdpInfo used only on line: 235

328:        uint256 beforeSwapsLength = operation.swapsBefore.length;	// @audit-issue: beforeSwapsLength used only on line: 329

346:        uint256 afterSwapsLength = operation.swapsAfter.length;	// @audit-issue: afterSwapsLength used only on line: 347

389:        LeverageMacroOperation memory leverageMacroData = abi.decode(data, (LeverageMacroOperation));	// @audit-issue: leverageMacroData used only on line: 390

424:        LeverageMacroOperation memory operation = decodeFLData(data);	// @audit-issue: operation used only on line: 426

464:        (bool success, ) = excessivelySafeCall(	// @audit-issue: success used only on line: 471
465:            swapData.addressForSwap,
466:            gasleft(),
467:            0,
468:            0,
469:            swapData.calldataForSwap
470:        );

541:        CloseCdpOperation memory flData = abi.decode(data, (CloseCdpOperation));	// @audit-issue: flData used only on line: 544
```
[143](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L143-L146), [232](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L232-L232), [328](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L328-L328), [346](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L346-L346), [389](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L389-L389), [424](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L424-L424), [464](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L464-L470), [541](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L541-L541), 


#### Recommendation

Eliminate single-use stack variables in Solidity to optimize gas consumption. Directly use the assigned value in the place of the variable. This approach saves the 3 gas typically used for the extra stack assignment, streamlining the function's execution and enhancing overall gas efficiency.

### `Internal` functions only called once can be inlined to save gas
If an internal function is only used once, there is no need to modularize it, unless the function calling it would otherwise be too long and complex. Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

132:    function _adjustCdpOperation(	// @audit-issue

165:    function _openCdpOperation(	// @audit-issue

200:    function _closeCdpOperation(	// @audit-issue

249:    function _getSwapChecks(address tokenToCheck, uint256 expectedMinOut) 	// @audit-issue
```
[132](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L132-L132), [165](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L165-L165), [200](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L200-L200), [249](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L249-L249), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

448:    function _doSwap(SwapOperation memory swapData) internal {	// @audit-issue

485:    function _doSwapChecks(SwapCheck[] memory swapChecks) internal {	// @audit-issue

500:    function _ensureNotSystem(address addy) internal {	// @audit-issue

510:    function _openCdpCallback(bytes memory data) internal virtual {	// @audit-issue

524:    function _openCdpForCallback(bytes memory data) internal virtual {	// @audit-issue

540:    function _closeCdpCallback(bytes memory data) internal virtual {	// @audit-issue

548:    function _adjustCdpCallback(bytes memory data) internal virtual {	// @audit-issue

562:    function _claimSurplusCallback() internal virtual {	// @audit-issue

568:    function excessivelySafeCall(	// @audit-issue
```
[448](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L448-L448), [485](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L485-L485), [500](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L500-L500), [510](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L510-L510), [524](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L524-L524), [540](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L540-L540), [548](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L548-L548), [562](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L562-L562), [568](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L568-L568), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

310:    function _requireNonZeroAdjustment(	// @audit-issue

321:    function _requireSingularMarginChange(	// @audit-issue
```
[310](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L310-L310), [321](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L321-L321), 


#### Recommendation

Inline 'internal' functions in Solidity that are called only once to save gas. This avoids the additional gas cost of 20 to 40 units associated with extra JUMP instructions and stack operations required for separate function calls, unless the calling function becomes too complex.

### `Private` functions only called once can be inlined to save gas
If a private function is only used once, there is no need to modularize it, unless the function calling it would otherwise be too long and complex. Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

88:    function _sweepStEth() private {	// @audit-issue

100:    function _getAdjustCdpParams(	// @audit-issue
```
[88](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L88-L88), [100](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L100-L100), 


#### Recommendation

Inline 'private' functions in Solidity that are called only once to save gas. This avoids the additional gas cost of 20 to 40 units associated with extra JUMP instructions and stack operations required for separate function calls, unless the calling function becomes too complex.

### Redundant State Variable Getters in Solidity
In Solidity, state variables can have different visibility levels, including `public`. When a state variable is declared as `public`, the Solidity compiler automatically generates a getter function for it. This implicit getter has the same name as the state variable and allows external callers to query the variable's value.

A common oversight is the explicit creation of a function that returns the value of a public state variable. This function essentially duplicates the functionality already provided by the automatically generated getter. For instance, if there's a public state variable `uint256 public value;`, there's no need for a function like `function getValue() public view returns (uint256) { return value; }`, as the compiler already provides a `value()` function.



```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

61:    function owner() public override returns (address) {	// @audit-issue
62:        return theOwner;
63:    }
```
[61](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L61-L63), 


#### Recommendation

Avoid creating explicit getter functions for 'public' state variables in Solidity. The compiler automatically generates getters for such variables, making additional functions redundant. This practice helps reduce contract size, lowers deployment costs, and simplifies maintenance and understanding of the contract.

### Multiple Pragma Definition
In Solidity, pragma statements are used to specify the compiler version and settings for a smart contract. This issue occurs when multiple pragma statements are defined within the same contract or file. Each pragma statement should be unique within a contract or file, as it sets the compiler version and potentially other compiler-specific configurations.


```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

2:pragma solidity 0.8.17;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L2-L2), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

2:pragma solidity ^0.8.17;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L2-L2), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

2:pragma solidity 0.8.17;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L2-L2), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

2:pragma solidity 0.8.17;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L2-L2), 


#### Recommendation

Ensure each Solidity contract or file contains a single, unique pragma statement to clearly specify the intended compiler version and settings. Avoid multiple pragma definitions to prevent confusion and potential compilation issues.

### Use `private` Rather than `public` for Constants 
In Solidity, constants represent immutable values that cannot be changed after they are set at compile-time. By default, constants have internal visibility, meaning they can be accessed within the contract they are declared in and in derived contracts. If a constant is explicitly declared as `public`, Solidity automatically generates a getter function for it. While this might seem harmless, it actually incurs a gas overhead, especially when the contract is deployed, as the EVM needs to generate bytecode for that getter. Conversely, declaring constants as `private` ensures that no additional getter is generated, optimizing gas usage.

```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

18:    address public constant NATIVE_ETH_ADDRESS = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;	// @audit-issue

19:    uint256 public constant LIQUIDATOR_REWARD = 2e17;	// @audit-issue

20:    uint256 public constant MIN_NET_STETH_BALANCE = 2e18;	// @audit-issue
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L18-L18), [19](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L19-L19), [20](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L20-L20), 


#### Recommendation

To optimize gas usage in your Solidity contracts, declare constants with `private` visibility rather than `public` when possible. Using `private` prevents the automatic generation of a getter function, reducing gas overhead, especially during contract deployment.

### Consider pre-calculating the address of `address(this)` to save gas
Use `foundry`'s [`script.sol`](https://book.getfoundry.sh/reference/forge-std/compute-create-address) or `solady`'s [`LibRlp.sol`](https://github.com/Vectorized/solady/blob/main/src/utils/LibRLP.sol) to save the value in a constant, which will avoid having to spend gas to push the value on the stack every time it's used.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

81:        uint256 bal = ebtcToken.balanceOf(address(this));	// @audit-issue

93:        uint256 bal = stEth.sharesOf(address(this));	// @audit-issue
```
[81](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L81-L81), [93](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L93-L93), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

294:        uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));	// @audit-issue

300:        uint256 _zapStEthBalanceAfter = stEth.balanceOf(address(this));	// @audit-issue

342:        uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));	// @audit-issue

360:        uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));	// @audit-issue

378:        uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));	// @audit-issue

396:        uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));	// @audit-issue

453:        uint256 _zapStEthBalanceDiff = stEth.balanceOf(address(this)) - _zapStEthBalanceBefore;	// @audit-issue
```
[294](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L294-L294), [300](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L300-L300), [342](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L342-L342), [360](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L360-L360), [378](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L378-L378), [396](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L396-L396), [453](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L453-L453), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

36:        uint256 _balBefore = stEth.balanceOf(address(this));	// @audit-issue

39:        uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;	// @audit-issue

44:        uint256 _wETHBalBefore = wrappedEth.balanceOf(address(this));	// @audit-issue

45:        wrappedEth.transferFrom(msg.sender, address(this), _initialWETH);	// @audit-issue

46:        uint256 _wETHReiceived = wrappedEth.balanceOf(address(this)) - _wETHBalBefore;	// @audit-issue

48:        uint256 _rawETHBalBefore = address(this).balance;	// @audit-issue

50:        uint256 _rawETHConverted = address(this).balance - _rawETHBalBefore;	// @audit-issue

61:            wstEth.transferFrom(msg.sender, address(this), _initialWstETH),	// @audit-issue

65:        uint256 _stETHBalBefore = stEth.balanceOf(address(this));	// @audit-issue

67:        uint256 _stETHReiceived = stEth.balanceOf(address(this)) - _stETHBalBefore;	// @audit-issue

109:        uint256 _balBefore = stEth.balanceOf(address(this));	// @audit-issue

110:        stEth.transferFrom(msg.sender, address(this), _initialStETH);	// @audit-issue

111:        uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;	// @audit-issue

122:                address(this),	// @audit-issue
```
[36](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L36-L36), [39](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L39-L39), [44](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L44-L44), [45](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L45-L45), [46](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L46-L46), [48](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L48-L48), [50](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L50-L50), [61](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L61-L61), [65](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L65-L65), [67](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L67-L67), [109](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L109-L109), [110](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L110-L110), [111](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L111-L111), [122](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L122-L122), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

135:                address(this),	// @audit-issue

179:                address(this),	// @audit-issue

187:                IERC3156FlashBorrower(address(this)),	// @audit-issue

194:                IERC3156FlashBorrower(address(this)),	// @audit-issue

253:        uint256 ebtcBal = ebtcToken.balanceOf(address(this));	// @audit-issue

254:        uint256 collateralBal = stETH.sharesOf(address(this));	// @audit-issue

402:        require(initiator == address(this), "LeverageMacroReference: wrong initiator for flashloan");	// @audit-issue

491:                    IERC20(swapChecks[i].tokenToCheck).balanceOf(address(this)) >	// @audit-issue

506:        require(addy != address(this)); // If it could call this it could fake the forwarded caller	// @audit-issue
```
[135](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L135-L135), [179](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L179-L179), [187](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L187-L187), [194](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L194-L194), [253](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L253-L253), [254](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L254-L254), [402](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L402-L402), [491](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L491-L491), [506](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L506-L506), 


#### Recommendation

To enhance gas efficiency, cache the contract's address by storing `address(this)` in a state variable at the point of contract deployment or initialization. Use this cached address throughout the contract instead of repeatedly calling `address(this)`. This practice reduces the gas cost associated with multiple computations of the contract's address, leading to more efficient contract execution, especially in scenarios with frequent usage of the contract's address.

### Counting down in for statements is more gas efficient
Looping downwards in Solidity is more gas efficient due to how the EVM compares variables. In a 'for' loop that counts down, the end condition is usually to compare with zero, which is cheaper than comparing with another number. As such, restructure your loops to count downwards where possible.

```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

438:                ++i;	// @audit-issue

488:            for (uint256 i; i < length; ++i) {	// @audit-issue
```
[438](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L438-L438), [488](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L488-L488), 


#### Recommendation

Where feasible, refactor `for` loops in your Solidity contracts to count downwards. Adjust the loop initialization, condition, and iteration statements to decrement the loop variable and terminate the loop when it reaches zero. This approach can lead to gas savings, making your contract more efficient in terms of execution costs. Ensure that this refactoring aligns with the logic and requirements of your contract, and thoroughly test to confirm that the revised loop behavior matches the intended functionality.

### Consider using solady's 'FixedPointMathLib'
Using Solady's "FixedPointMathLib" for multiplication or division operations in Solidity can lead to significant gas savings. This library is designed to optimize fixed-point arithmetic operations, which are common in financial calculations involving tokens or currencies. By implementing more efficient algorithms and assembly optimizations, "FixedPointMathLib" minimizes the computational resources required for these operations. For contracts that frequently perform such calculations, integrating this library can reduce transaction costs, thereby enhancing overall performance and cost-effectiveness. However, developers must ensure compatibility with their existing codebase and thoroughly test for accuracy and expected behavior to avoid any unintended consequences.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

268:                    value:  _totalCollateral * _collValidationBuffer / BPS,	// @audit-issue

288:            IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData.eBTCToMint * zapFeeBPS / BPS);	// @audit-issue

309:                IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData._EBTCChange * zapFeeBPS / BPS);	// @audit-issue
```
[268](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L268-L268), [288](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L288-L288), [309](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L309-L309), 


#### Recommendation

Consider integrating Solady's 'FixedPointMathLib' into your Solidity contracts for optimized fixed-point arithmetic operations. This library can provide substantial gas savings and enhance the performance of your contract. Before integration, evaluate how 'FixedPointMathLib' aligns with your contract’s requirements. Ensure thorough testing for accuracy and compatibility with your existing contract logic. Carefully document any changes and keep track of how these optimizations affect your contract's operations to maintain transparency and reliability in your application. Adopting 'FixedPointMathLib' should be a considered decision, balancing the benefits of gas efficiency with the need for maintaining code clarity and functionality.

### Reduce Gas Usage by Moving to Solidity 0.8.19 or Later
This issue highlights the opportunity to reduce gas consumption in a smart contract by upgrading the Solidity compiler version to 0.8.19 or a later release. Gas usage is a critical consideration in Ethereum smart contracts, as it directly affects transaction costs and contract execution efficiency. Newer compiler versions often come with optimizations and improvements that can lead to reduced gas costs for certain operations and contract structures.


```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

2:pragma solidity 0.8.17;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L2-L2), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

2:pragma solidity ^0.8.17;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L2-L2), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

2:pragma solidity 0.8.17;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L2-L2), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

2:pragma solidity 0.8.17;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L2-L2), 


#### Recommendation

Consider upgrading to Solidity version 0.8.19 or later to leverage compiler optimizations for reduced gas consumption. Newer versions often include improvements that enhance transaction cost-efficiency and overall contract execution.

### Constant Keccak Variables Are Treated As Expressions, Not Constants
In Solidity, state variables or local variables declared with the `constant` keyword are expected to represent constant values that are determined at compile time. These constants typically do not incur any gas costs when accessed since their values are hard-coded into the contract bytecode.

However, this expected behavior deviates when using the `keccak256()` function. When a variable is initialized with a `keccak256()` hash computation, even if declared as `constant`, it isn't truly treated as a constant in the resulting bytecode. Instead, every time this "constant" is referenced, the EVM recalculates the hash. This behavior contrasts with other true constants, which are embedded directly into the bytecode and accessed without additional computations.


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

37:    bytes32 constant FLASH_LOAN_SUCCESS = keccak256("ERC3156FlashBorrower.onFlashLoan");	// @audit-issue
```
[37](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L37-L37), 


#### Recommendation

Avoid using 'keccak256()' to initialize 'constant' variables in Solidity. Instead, precompute the hash value outside of the contract and assign this precomputed value to the constant. This ensures the variable is a true constant, eliminating redundant hash computations and saving gas.

### Using `bool`s for storage incurs overhead
[Booleans are more expensive than uint256](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27) or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.
Use `uint256(0)` and `uint256(1)` for true/false to avoid a Gwarmaccess (**[100 gas](https://gist.github.com/IllIllI000/1b70014db712f8572a72378321250058)**) for the extra SLOAD.


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

35:    bool internal immutable willSweep;	// @audit-issue
```
[35](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L35-L35), 


#### Recommendation

To minimize gas overhead in your Solidity contracts, consider using `uint256(1)` and `uint256(2)` to represent `true` and `false`, respectively, instead of `bool` types for storage. This approach avoids additional `SLOAD` and `SSTORE` operations, resulting in more gas-efficient code.

### Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Each operation involving a `uint8` costs an extra [22-28](https://gist.github.com/IllIllI000/9388d20c70f9a4632eb3ca7836f54977) gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

572:        uint16 _maxCopy,	// @audit-issue
```
[572](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L572-L572), 


#### Recommendation

Minimize gas overhead by using 'uint256' or 'int256' instead of smaller integer types in Solidity contracts. The EVM operates more efficiently with 32-byte sizes. Downcast to smaller types only when necessary, as operations with smaller types like 'uint8' incur extra gas due to additional EVM operations for size adjustment.

### Avoid Unnecessary Public Variables
Public state variables in Solidity automatically generate getter functions, increasing contract size and potentially leading to higher deployment and interaction costs. To optimize gas usage and contract efficiency, minimize the use of public variables unless external access is necessary. Instead, use internal or private visibility combined with explicit getter functions when required. This practice not only reduces contract size but also provides better control over data access and manipulation, enhancing security and readability. Prioritize lean, efficient contracts to ensure cost-effectiveness and better performance on the blockchain.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

18:    uint256 internal constant PRECISION = 1e18;	// @audit-issue

19:    uint256 internal constant BPS = 10000;	// @audit-issue

20:	// @audit-issue

21:    address public immutable theOwner;	// @audit-issue

23:    uint256 public immutable zapFeeBPS;	// @audit-issue

24:    address public immutable zapFeeReceiver;	// @audit-issue

25:	// @audit-issue

29:        ZapRouterBase(	// @audit-issue

30:            params.borrowerOperations, 	// @audit-issue

31:            IERC20(params.wstEth), 	// @audit-issue

32:            IERC20(params.weth), 	// @audit-issue

33:            IStETH(params.stEth)	// @audit-issue

34:        )	// @audit-issue

11:import {ZapRouterBase} from "./ZapRouterBase.sol";	// @audit-issue

22:    address public immutable DEX;	// @audit-issue
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L18-L18), [19](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L19-L19), [20](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L20-L20), [21](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L21-L21), [23](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L23-L23), [24](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L24-L24), [25](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L25-L25), [29](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L29-L29), [30](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L30-L30), [31](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L31-L31), [32](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L32-L32), [33](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L33-L33), [34](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L34-L34), [11](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L11-L11), [22](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L22-L22), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

18:contract EbtcLeverageZapRouter is LeverageZapRouterBase {	// @audit-issue

19:    using SafeERC20 for IERC20;	// @audit-issue

20:	// @audit-issue

21:    constructor(	// @audit-issue

23:    ) LeverageZapRouterBase(params) { }	// @audit-issue

24:	// @audit-issue

25:    /// @dev Open a CDP with raw native Ether	// @audit-issue

29:    /// @param _stEthLoanAmount The flash loan amount needed to open the leveraged Cdp position	// @audit-issue

30:    /// @param _ethMarginBalance The amount of margin deposit (converted from raw Ether) from the user, higher margin equals lower CR	// @audit-issue

31:    /// @param _stEthDepositAmount The total stETH collateral amount deposited (added) for the specified Cdp	// @audit-issue

32:    /// @param _positionManagerPermit PositionPermit required for Zap approved by calling user	// @audit-issue

33:    /// @param _tradeData DEX calldata for converting between debt and collateral	// @audit-issue

34:    function openCdpWithEth(	// @audit-issue

11:import {SafeERC20} from "@ebtc/contracts/Dependencies/SafeERC20.sol";	// @audit-issue

22:        IEbtcLeverageZapRouter.DeploymentParams memory params	// @audit-issue
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L18-L18), [19](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L19-L19), [20](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L20-L20), [21](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L21-L21), [23](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L23-L23), [24](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L24-L24), [25](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L25-L25), [29](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L29-L29), [30](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L30-L30), [31](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L31-L31), [32](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L32-L32), [33](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L33-L33), [34](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L34-L34), [11](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L11-L11), [22](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L22-L22), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

18:    address public constant NATIVE_ETH_ADDRESS = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;	// @audit-issue

19:    uint256 public constant LIQUIDATOR_REWARD = 2e17;	// @audit-issue

20:    uint256 public constant MIN_NET_STETH_BALANCE = 2e18;	// @audit-issue

21:    uint256 public immutable MIN_CHANGE;	// @audit-issue

23:    IERC20 public immutable wstEth;	// @audit-issue

24:    IStETH public immutable stEth;	// @audit-issue

25:    IERC20 public immutable wrappedEth;	// @audit-issue
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L18-L18), [19](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L19-L19), [20](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L20-L20), [21](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L21-L21), [23](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L23-L23), [24](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L24-L24), [25](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L25-L25), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

29:    IBorrowerOperations public immutable borrowerOperations;	// @audit-issue

30:    IActivePool public immutable activePool;	// @audit-issue

31:    ICdpCdps public immutable cdpManager;	// @audit-issue

32:    IEBTCToken public immutable ebtcToken;	// @audit-issue

33:    ISortedCdps public immutable sortedCdps;	// @audit-issue

34:    ICollateralToken public immutable stETH;	// @audit-issue
```
[29](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L29-L29), [30](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L30-L30), [31](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L31-L31), [32](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L32-L32), [33](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L33-L33), [34](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L34-L34), 


#### Recommendation

Avoid creating explicit getter functions for 'public' state variables in Solidity. The compiler automatically generates getters for such variables, making additional functions redundant. This practice helps reduce contract size, lowers deployment costs, and simplifies maintenance and understanding of the contract.

### Use `do while` loops intead of for loops
A `do while` loop will cost less gas since the condition is not being checked for the first iteration.
```solidity
uint256 i = 1;
do {                   
    param2 += i;
    i++;
}
while (i < 50);
``` 
is better than
```solidity
for(uint256 i = 1; i< 50; i++){
    param1 += i;
}
```


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

435:        for (uint256 i; i < swapLength; ) {	// @audit-issue

488:            for (uint256 i; i < length; ++i) {	// @audit-issue
```
[435](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L435-L435), [488](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L488-L488), 


#### Recommendation

Where appropriate, consider using a `do while` loop instead of a `for` loop in your Solidity contracts. This is especially beneficial when the first iteration of the loop does not require a condition check. Refactor your loop logic to fit the `do while` structure for more gas-efficient execution. However, ensure that the loop's logic and termination conditions are correctly implemented to avoid infinite loops or other logical errors. Always balance gas efficiency with code readability and the specific requirements of your contract's logic.

### Using XOR (^) and AND (&) bitwise equivalents for gas optimizations
Given 4 variables a, b, c and d represented as such:
```
0 0 0 0 0 1 1 0 <- a
0 1 1 0 0 1 1 0 <- b
0 0 0 0 0 0 0 0 <- c
1 1 1 1 1 1 1 1 <- d
```
To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

114:            msg.sender == address(wrappedEth),	// @audit-issue

282:        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for close!");	// @audit-issue

326:            _stEthMarginIncrease == 0 || _stEthMarginDecrease == 0,	// @audit-issue

410:        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for adjust!");	// @audit-issue
```
[114](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L114-L114), [282](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L282-L282), [326](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L326-L326), [410](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L410-L410), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

55:        require(msg.value == _initialETH, "EbtcZapRouter: Incorrect ETH amount");	// @audit-issue

137:            _change == 0 || _change >= MIN_CHANGE,	// @audit-issue
```
[55](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L55-L55), [137](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L137-L137), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

45:        require(owner() == msg.sender, "Must be owner");	// @audit-issue

131:            operation.operationType == OperationType.OpenCdpOperation &&	// @audit-issue

140:            operation.operationType == OperationType.OpenCdpForOperation &&	// @audit-issue

185:        if (flType == FlashLoanType.eBTC) {	// @audit-issue

192:        } else if (flType == FlashLoanType.stETH) {	// @audit-issue

207:        if (postCheckType == PostOperationCheck.openCdp) {	// @audit-issue

213:                cdpInfo.status == checkParams.expectedStatus,	// @audit-issue

219:        if (postCheckType == PostOperationCheck.cdpStats) {	// @audit-issue

225:                cdpInfo.status == checkParams.expectedStatus,	// @audit-issue

231:        if (postCheckType == PostOperationCheck.isClosed) {	// @audit-issue

235:                cdpInfo.status == checkParams.expectedStatus,	// @audit-issue

278:        if (check.operator == Operator.skip) {	// @audit-issue

281:        } else if (check.operator == Operator.gte) {	// @audit-issue

283:        } else if (check.operator == Operator.lte) {	// @audit-issue

285:        } else if (check.operator == Operator.equal) {	// @audit-issue

286:            require(check.value == valueToCheck, "!LeverageMacroReference: equal post check");	// @audit-issue

334:        if (operation.operationType == OperationType.OpenCdpOperation) {	// @audit-issue

336:        } else if (operation.operationType == OperationType.OpenCdpForOperation) {	// @audit-issue

338:        } else if (operation.operationType == OperationType.CloseCdpOperation) {	// @audit-issue

340:        } else if (operation.operationType == OperationType.AdjustCdpOperation) {	// @audit-issue

342:        } else if (operation.operationType == OperationType.ClaimSurplusOperation) {	// @audit-issue

402:        require(initiator == address(this), "LeverageMacroReference: wrong initiator for flashloan");	// @audit-issue

405:        if (token == address(ebtcToken)) {	// @audit-issue

407:                msg.sender == address(borrowerOperations),	// @audit-issue

413:                msg.sender == address(activePool),	// @audit-issue
```
[45](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L45-L45), [131](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L131-L131), [140](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L140-L140), [185](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L185-L185), [192](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L192-L192), [207](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L207-L207), [213](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L213-L213), [219](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L219-L219), [225](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L225-L225), [231](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L231-L231), [235](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L235-L235), [278](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L278-L278), [281](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L281-L281), [283](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L283-L283), [285](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L285-L285), [286](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L286-L286), [334](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L334-L334), [336](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L336-L336), [338](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L338-L338), [340](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L340-L340), [342](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L342-L342), [402](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L402-L402), [405](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L405-L405), [407](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L407-L407), [413](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L413-L413), 


#### Recommendation

Review your Solidity contracts to identify any computations that are performed multiple times with the same inputs. Cache the results of these computations in local variables and reuse them within the function or across function calls if the state remains unchanged.

### The result of a function call should be cached rather than re-calling the function
The function calls in solidity are expensive. If the same result of the same function calls are to be used several times, the result should be cached to reduce the gas consumption of repeated calls.        

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

57:        stETH.approve(address(activePool), type(uint256).max);	// @audit-issue: Function call is called multiple times at line(s) [58, 56, 55].
```
[57](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L57-L57), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

300:        uint256 _zapStEthBalanceAfter = stEth.balanceOf(address(this));	// @audit-issue: Function call `balanceOf` is called multiple times at lines [294].
```
[300](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L300-L300), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

39:        uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;	// @audit-issue: Function call `balanceOf` is called multiple times at lines [36].

46:        uint256 _wETHReiceived = wrappedEth.balanceOf(address(this)) - _wETHBalBefore;	// @audit-issue: Function call `balanceOf` is called multiple times at lines [44].

65:        uint256 _stETHBalBefore = stEth.balanceOf(address(this));	// @audit-issue: Function call `balanceOf` is called multiple times at lines [67].

109:        uint256 _balBefore = stEth.balanceOf(address(this));	// @audit-issue: Function call `balanceOf` is called multiple times at lines [111].
```
[39](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L39-L39), [46](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L46-L46), [65](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L65-L65), [109](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L109-L109), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

137:                sortedCdps.nextCdpNonce()	// @audit-issue: Function call `nextCdpNonce` is called multiple times at lines [151].

187:                IERC3156FlashBorrower(address(this)),	// @audit-issue: Function call `IERC3156FlashBorrower` is called multiple times at lines [194].

190:                abi.encode(operation)	// @audit-issue: Function call `encode` is called multiple times at lines [197].

222:            _doCheckValueType(checkParams.expectedDebt, cdpInfo.debt);	// @audit-issue: Function call `_doCheckValueType` is called multiple times at lines [210].

223:            _doCheckValueType(checkParams.expectedCollateral, cdpInfo.coll);	// @audit-issue: Function call `_doCheckValueType` is called multiple times at lines [211].

220:            ICdpManagerData.Cdp memory cdpInfo = cdpManager.Cdps(checkParams.cdpId);	// @audit-issue: Function call `Cdps` is called multiple times at lines [232].

477:        IERC20(swapData.tokenForSwap).safeApprove(swapData.addressForApprove, 0);	// @audit-issue: Function call `IERC20` is called multiple times at lines [455].
```
[137](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L137-L137), [187](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L187-L187), [190](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L190-L190), [222](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L222-L222), [223](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L223-L223), [220](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L220-L220), [477](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L477-L477), 


#### Recommendation

Cache the result of function calls in Solidity instead of making repeated calls to the same function. This practice significantly reduces gas consumption by minimizing costly function call operations.

### Avoid zero transfers calls
In Solidity, unnecessary operations can waste gas. For example, a transfer function without a zero amount check uses gas even if called with a zero amount, since the contract state remains unchanged. Implementing a zero amount check avoids these unnecessary function calls, saving gas and improving efficiency.

```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

45:        wrappedEth.transferFrom(msg.sender, address(this), _initialWETH);	// @audit-issue

61:            wstEth.transferFrom(msg.sender, address(this), _initialWstETH),	// @audit-issue

103:            stEth.transfer(msg.sender, _stEthVal);	// @audit-issue

110:        stEth.transferFrom(msg.sender, address(this), _initialStETH);	// @audit-issue
```
[45](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L45-L45), [61](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L61-L61), [103](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L103-L103), [110](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L110-L110), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

270:        IERC20(token).safeTransfer(msg.sender, amount);	// @audit-issue
```
[270](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L270-L270), 


#### Recommendation

Include a condition in your transfer functions to check for and prevent zero-value transfers. Implement a `require(amount > 0, "Transfer amount must be greater than zero");` statement at the beginning of the function. This preemptive check ensures that the function only proceeds with non-zero transfer amounts, avoiding wasteful operations and saving gas. Apply this optimization across all functions involving token transfers or similar operations to improve your contract's gas efficiency and operational effectiveness.

### Empty Blocks Should Be Removed Or Emit Something
The code should be refactored such that empty blocks no longer exist, or the block should do something useful, such as emitting an event or reverting. Empty blocks can introduce confusion and potential errors when the code is modified in the future.


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

129:        {} catch {	// @audit-issue

129:        {} catch {	// @audit-issue
130:            /// @notice adding try...catch around to mitigate potential permit front-running
131:            /// see: https://www.trust-security.xyz/post/permission-denied
132:        }
```
[129](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L129-L129), [129](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L129-L132), 


#### Recommendation

Remove empty blocks in Solidity code to avoid confusion and potential errors. If a block is necessary, ensure it performs a useful action, like emitting an event or reverting, to justify its presence and clarify its purpose.

### `abi.encode()` is less efficient than `abi.encodePacked()`
When working with Solidity, it is important to pay attention to gas efficiency in order to optimize smart contracts. In this blog post, we will compare the gas costs of `abi.encode()` and `abi.encodepacked()` and explore why the latter is more efficient.Source: [reference](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)


```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

108:        op.OperationData = abi.encode(_cdp);	// @audit-issue

177:        op.OperationData = abi.encode(_cdp);	// @audit-issue

212:        op.OperationData = abi.encode(cdp);	// @audit-issue
```
[108](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L108-L108), [177](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L177-L177), [212](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L212-L212), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

190:                abi.encode(operation)	// @audit-issue

197:                abi.encode(operation)	// @audit-issue
```
[190](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L190-L190), [197](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L197-L197), 


#### Recommendation

Optimize gas usage by preferring 'abi.encodePacked()' over 'abi.encode()' where appropriate, as 'abi.encodePacked()' is generally more gas-efficient. However, ensure its suitability based on the data type and structure requirements, as 'abi.encodePacked()' can lead to ambiguity in certain cases.

### `x += y` costs more gas than `x = x + y` for stack variables
Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

425:            marginDecrease += _params.stEthMarginBalance;	// @audit-issue

430:            marginIncrease += _params.stEthMarginBalance;	// @audit-issue
```
[425](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L425-L425), [430](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L430-L430), 


#### Recommendation

Prefer using 'x = x + y' over 'x += y' for state variable assignments in Solidity to save gas. The latter incurs extra costs due to additional JUMP instructions and stack operations associated with non-inlined function calls.

### Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

26:    constructor(
27:        IEbtcLeverageZapRouter.DeploymentParams memory params	// @audit-issue
28:    )

100:    function _getAdjustCdpParams(
101:        AdjustCdpOperation memory _cdp,	// @audit-issue
102:        TradeData calldata _tradeData
103:    ) private view returns (LeverageMacroOperation memory op) {

132:    function _adjustCdpOperation(
133:        bytes32 _cdpId,
134:        FlashLoanType _flType,
135:        uint256 _flAmount,
136:        AdjustCdpOperation memory _cdp,	// @audit-issue
137:        uint256 debt,
138:        uint256 coll,
139:        TradeData calldata _tradeData
140:    ) internal {

165:    function _openCdpOperation(
166:        bytes32 _cdpId,
167:        OpenCdpForOperation memory _cdp,	// @audit-issue
168:        uint256 _flAmount,
169:        TradeData calldata _tradeData
170:    ) internal {

276:    function _openCdpForCallback(bytes memory data) internal override {	// @audit-issue

294:    function _adjustCdpCallback(bytes memory data) internal override {	// @audit-issue
```
[27](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L26-L28), [101](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L100-L103), [136](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L132-L140), [167](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L165-L170), [276](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L276-L276), [294](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L294-L294), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

21:    constructor(
22:        IEbtcLeverageZapRouter.DeploymentParams memory params	// @audit-issue
23:    ) LeverageZapRouterBase(params) { }

336:    function adjustCdpWithEth(
337:        bytes32 _cdpId,
338:        AdjustCdpParams memory _params,	// @audit-issue
339:        bytes calldata _positionManagerPermit,
340:        TradeData calldata _tradeData
341:    ) external payable {

354:    function adjustCdpWithWstEth(
355:        bytes32 _cdpId,
356:        AdjustCdpParams memory _params,	// @audit-issue
357:        bytes calldata _positionManagerPermit,
358:        TradeData calldata _tradeData
359:    ) external {

372:    function adjustCdpWithWrappedEth(
373:        bytes32 _cdpId,
374:        AdjustCdpParams memory _params,	// @audit-issue
375:        bytes calldata _positionManagerPermit,
376:        TradeData calldata _tradeData
377:    ) external {

390:    function adjustCdp(
391:        bytes32 _cdpId,
392:        AdjustCdpParams memory _params,	// @audit-issue
393:        bytes calldata _positionManagerPermit,
394:        TradeData calldata _tradeData
395:    ) external {

403:    function _adjustCdp(
404:        bytes32 _cdpId,
405:        AdjustCdpParams memory _params,	// @audit-issue
406:        bytes calldata _positionManagerPermit,
407:        TradeData calldata _tradeData,
408:        uint256 _zapStEthBalanceBefore
409:    ) internal nonReentrant {
```
[22](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L21-L23), [338](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L336-L341), [356](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L354-L359), [374](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L372-L377), [392](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L390-L395), [405](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L403-L409), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

115:    function _permitPositionManagerApproval(
116:        IBorrowerOperations borrowerOperations,
117:        PositionManagerPermit memory _positionManagerPermit	// @audit-issue
118:    ) internal {
```
[117](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L115-L118), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

167:    function _doOperation(
168:        FlashLoanType flType,
169:        uint256 borrowAmount,
170:        LeverageMacroOperation memory operation,	// @audit-issue
171:        PostOperationCheck postCheckType,
172:        PostCheckParams memory checkParams,
173:        bytes32 expectedCdpId
174:    ) internal {

167:    function _doOperation(
168:        FlashLoanType flType,
169:        uint256 borrowAmount,
170:        LeverageMacroOperation memory operation,
171:        PostOperationCheck postCheckType,
172:        PostCheckParams memory checkParams,	// @audit-issue
173:        bytes32 expectedCdpId
174:    ) internal {

277:    function _doCheckValueType(CheckValueAndType memory check, uint256 valueToCheck) internal {	// @audit-issue

327:    function _handleOperation(LeverageMacroOperation memory operation) internal {	// @audit-issue

432:    function _doSwaps(SwapOperation[] memory swapData) internal {	// @audit-issue

448:    function _doSwap(SwapOperation memory swapData) internal {	// @audit-issue

485:    function _doSwapChecks(SwapCheck[] memory swapChecks) internal {	// @audit-issue

510:    function _openCdpCallback(bytes memory data) internal virtual {	// @audit-issue

524:    function _openCdpForCallback(bytes memory data) internal virtual {	// @audit-issue

540:    function _closeCdpCallback(bytes memory data) internal virtual {	// @audit-issue

548:    function _adjustCdpCallback(bytes memory data) internal virtual {	// @audit-issue

568:    function excessivelySafeCall(
569:        address _target,
570:        uint256 _gas,
571:        uint256 _value,
572:        uint16 _maxCopy,
573:        bytes memory _calldata	// @audit-issue
574:    ) internal returns (bool, bytes memory) {
```
[170](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L167-L174), [172](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L167-L174), [277](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L277-L277), [327](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L327-L327), [432](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L432-L432), [448](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L448-L448), [485](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L485-L485), [510](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L510-L510), [524](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L524-L524), [540](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L540-L540), [548](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L548-L548), [573](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L568-L574), 


#### Recommendation

To optimize gas usage in your Solidity functions, mark data types as `calldata` instead of `memory` wherever applicable. This prevents unnecessary data loading into memory. Use `calldata` for function arguments that do not require changes within the function, except when passing them into another function that explicitly requires `memory` storage.

### Nesting `if` statements that uses `&&` saves gas
In Solidity, the way conditional checks are structured can impact the gas consumption of a transaction. When conditions are combined using `&&` within an `if` statement, Solidity short-circuits the evaluation, meaning that if the first condition is `false`, the subsequent conditions won't be evaluated. This behavior can lead to gas savings compared to using separate nested `if` statements because not all conditions might need to be checked every time. By efficiently structuring these conditional checks, contracts can optimize the gas required for execution, leading to reduced costs for users.


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

343:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue

361:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue

379:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue

397:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue

424:        if (!_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue

429:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue
```
[343](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L343-L343), [361](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L361-L361), [379](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L379-L379), [397](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L397-L397), [424](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L424-L424), [429](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L429-L429), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

131:            operation.operationType == OperationType.OpenCdpOperation &&	// @audit-issue
132:            postCheckType != PostOperationCheck.none

140:            operation.operationType == OperationType.OpenCdpForOperation &&	// @audit-issue
141:            postCheckType != PostOperationCheck.none
```
[131](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L131-L132), [140](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L140-L141), 


#### Recommendation


When multiple conditions need to be checked successively, try to combine them in a single `if` statement using `&&` instead of nesting separate `if` statements. This will leverage short-circuit evaluation for potential gas savings.


### Use `!= 0` Instead of `> 0` for Unsigned Integer Comparison
In Solidity, unsigned integers (e.g., `uint256`, `uint8`, etc.) represent non-negative whole numbers, ranging from 0 to a maximum value determined by their bit size. When performing comparisons on these numbers, especially to check if they are non-zero, developers have options. A common practice is to compare against zero using the `>` operator, as in `value > 0`. However, given the nature of unsigned integers, a more straightforward and slightly gas-efficient comparison is to use the `!=` operator, as in `value != 0`.

The primary rationale is that the `!=` comparison directly checks for non-equality, whereas the `>` comparison checks if one value is strictly greater than another. For unsigned integers, where negative values don't exist, the `!= 0` check is both semantically clearer and potentially more efficient at the EVM bytecode level.


```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

45:        if (params.zapFeeBPS > 0) {	// @audit-issue

83:        if (bal > 0) {	// @audit-issue

95:        if (bal > 0) {	// @audit-issue

142:        uint256 newColl = _cdp._stEthBalanceIncrease > 0 ? 	// @audit-issue

277:        if (zapFeeBPS > 0) {	// @audit-issue

295:        if (zapFeeBPS > 0) {	// @audit-issue
```
[45](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L45-L45), [83](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L83-L83), [95](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L95-L95), [142](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L142-L142), [277](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L277-L277), [295](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L295-L295), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

222:        if (_positionManagerPermit.length > 0) {	// @audit-issue

289:        if (_positionManagerPermit.length > 0) {	// @audit-issue

316:            _stEthBalanceIncrease > 0 || _stEthBalanceDecrease > 0 || _debtChange > 0,	// @audit-issue

343:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue

361:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue

379:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue

397:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue

418:        if (_positionManagerPermit.length > 0) {	// @audit-issue
```
[222](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L222-L222), [289](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L289-L289), [316](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L316-L316), [343](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L343-L343), [361](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L361-L361), [379](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L379-L379), [397](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L397-L397), [418](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L418-L418), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

176:        if (operation.amountToTransferIn > 0) {	// @audit-issue

256:        if (ebtcBal > 0) {	// @audit-issue

329:        if (beforeSwapsLength > 0) {	// @audit-issue
```
[176](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L176-L176), [256](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L256-L256), [329](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L329-L329), 


#### Recommendation

Use '!= 0' instead of '> 0' for comparing unsigned integers in Solidity. This is semantically clearer and slightly more gas-efficient, as it directly checks for non-zero values without considering unnecessary relational comparisons.

### Constructor Can Be Marked As Payable
`payable` functions cost less gas to execute, since the compiler does not have to add extra checks to ensure that a payment wasn't provided.

A `constructor` can safely be marked as `payable`, since only the deployer would be able to pass funds, and the project itself would not pass any funds.



```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

26:    constructor(	// @audit-issue
```
[26](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L26-L26), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

21:    constructor(	// @audit-issue
```
[21](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L21-L21), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

27:    constructor(address _borrowerOperations, IERC20 _wstEth, IERC20 _wEth, IStETH _stEth) {	// @audit-issue
```
[27](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L27-L27), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

51:    constructor(	// @audit-issue
```
[51](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L51-L51), 


#### Recommendation

Mark constructors as 'payable' in Solidity contracts to reduce gas costs, as this eliminates the need for the compiler to add checks against incoming payments. This is safe because only the deployer can send funds during contract creation, and typically no funds are sent at this stage.

### Use `selfbalance()` instead of `address(this).balance`
In Solidity, contracts often need to query their own Ether balance. While `address(this).balance` has been a widely used approach to retrieve the current contract's balance, it is not the most gas-efficient method, especially with the introduction of the `SELFBALANCE` opcode in more recent EVM versions.

The `SELFBALANCE` opcode provides a more gas-efficient way to obtain the balance of the current contract. In Solidity, this can be invoked using the `selfbalance()` function. Compared to the `BALANCE` opcode used by `address(this).balance`, `SELFBALANCE` offers significant gas savings:

- `BALANCE`: The static gas cost is 0. If the accessed address is warm, the dynamic gas cost is 100. If the address is cold, the dynamic cost is 2,600.
  
- `SELFBALANCE`: Semantically equivalent to the `BALANCE` opcode when called on the contract's own address, but with a dramatically reduced minimum gas cost of 5.

Switching to `selfbalance()` can lead to significant gas savings, especially in operations or functions that frequently check the contract's balance.


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

48:        uint256 _rawETHBalBefore = address(this).balance;	// @audit-issue

50:        uint256 _rawETHConverted = address(this).balance - _rawETHBalBefore;	// @audit-issue
```
[48](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L48-L48), [50](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L50-L50), 


#### Recommendation

To optimize gas usage when querying a contract's own Ether balance in Solidity, it's recommended to use the `selfbalance()` function, which utilizes the more gas-efficient `SELFBALANCE` opcode. This can result in significant gas savings compared to `address(this).balance`.

### Optimize names to save gas
`public`/`external` function names and `public` member variable names can be optimized to save gas. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).


```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

15:abstract contract LeverageZapRouterBase is ZapRouterBase, LeverageMacroBase, ReentrancyGuard, IEbtcLeverageZapRouter {	// @audit-issue
```
[15](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L15-L15), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

18:contract EbtcLeverageZapRouter is LeverageZapRouterBase {	// @audit-issue
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L18-L18), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

13:interface IMinChangeGetter {	// @audit-issue

17:abstract contract ZapRouterBase is IEbtcZapRouterBase {	// @audit-issue
```
[13](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L13-L13), [17](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L17-L17), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

14:interface ICdpCdps {	// @audit-issue

26:abstract contract LeverageMacroBase {	// @audit-issue
```
[14](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L14-L14), [26](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L26-L26), 


#### Recommendation

Optimize gas usage by renaming 'public'/'external' functions and 'public' member variables in Solidity. Aim for shorter and more efficient names, especially for frequently called functions. This can save gas during deployment and reduce gas costs per call due to lower method ID sorting positions.

### Use `uint256(1)`/`uint256(2)` instead of `true`/`false` to save gas for changes
Avoids a Gsset (**20000 gas**) when changing from `false` to `true`, after having been `true` in the past. Since most of the bools aren't changed twice in one transaction, I've counted the amount of gas as half of the full amount, for each variable. Note that public state variables can be re-written to be `private` and use `uint256`, but have public getters returning `bool`s.

```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

35:    bool internal immutable willSweep;	// @audit-issue
```
[35](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L35-L35), 


#### Recommendation

To minimize gas overhead in your Solidity contracts, consider using `uint256(1)` and `uint256(2)` to represent `true` and `false`, respectively, instead of `bool` types for storage. This approach avoids additional `SLOAD` and `SSTORE` operations, resulting in more gas-efficient code.

### Consider activating `via-ir` for deploying
The IR-based code generator was developed to make code generation more performant by enabling optimization passes that can be applied across functions.

It is possible to activate the IR-based code generator through the command line by using the flag `--via-ir`or by including the option `{"viaIR": true}`.

Keep in mind that compiling with this option may take longer. However, you can simply test it before deploying your code. If you find that it provides better performance, you can add the `--via-ir` flag to your deploy command.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L1), 


#### Recommendation

Consider activating `via-ir`.

### Optimize Deployment Size by Fine-tuning IPFS Hash
The Solidity compiler appends 53 bytes of metadata to the smart contract code, incurring an extra cost of 10,600 gas. This additional expense arises from 200 gas per bytecode, plus calldata cost, which amounts to 16 gas for non-zero bytes and 4 gas for zero bytes. This results in a maximum of 848 extra gas in calldata cost.

Reducing this cost is crucial for the following reasons:

The metadata's 53-byte addition leads to a deployment cost increase of 10,600 gas. It can also result in an additional calldata cost of up to 848 gas. Ways to Minimize Gas Consumption:

Employ the `--no-cbor-metadata` compiler option to exclude metadata. Be cautious as this might impact contract verification. Search for code comments that yield an IPFS hash with more zeros, thereby reducing calldata costs.

```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L1), 


#### Recommendation

To optimize deployment size and reduce associated costs, consider the following strategies:
1. **Exclude Metadata with Compiler Option**: Use the `solc` compiler’s `--metadata-hash none` or `--no-cbor-metadata` option to prevent the inclusion of metadata in the compiled bytecode. This action reduces the bytecode size, thus lowering deployment gas costs. However, exercise caution with this approach, as it might affect the ability to verify the contract on platforms like Etherscan.

2. **Optimize IPFS Hash for More Zeros**: If excluding metadata is not desirable, another approach involves optimizing code comments or elements that influence the metadata hash generation to achieve an IPFS hash with a higher proportion of zeros. Since calldata costs are lower for zero bytes, a metadata hash with more zeros can reduce the calldata costs associated with contract interactions.

Example for excluding metadata:
```bash
solc --metadata-hash none YourContract.sol
```


### Low level `call` can be optimized with assembly
`returnData` is copied to memory even if the variable is not utilized: the proper way to handle this is through a low level assembly call.
```solidity
// before
(bool success,) = payable(receiver).call{gas: gas, value: value}("");

//after
bool success;
assembly {
    success := call(gas, receiver, value, 0, 0, 0, 0)
}
```


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

38:        payable(address(stEth)).call{value: _initialETH}("");	// @audit-issue
```
[38](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L38-L38), 


#### Recommendation

Optimize low-level 'call' operations in Solidity using assembly, especially when 'returnData' is not needed. This avoids unnecessary copying to memory and can lead to gas savings. For example, replace '(bool success,) = payable(receiver).call{gas: gas, value: value}("");' with an assembly block: 'assembly { success := call(gas, receiver, value, 0, 0, 0, 0) }'. This ensures a more efficient execution.

### Assembly: Use scratch space for building calldata
If an external call's calldata can fit into two or fewer words, use the scratch space to build the calldata, rather than allowing Solidity to do a memory expansion.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

55:        ebtcToken.approve(address(borrowerOperations), type(uint256).max);	// @audit-issue

56:        stETH.approve(address(borrowerOperations), type(uint256).max);	// @audit-issue

57:        stETH.approve(address(activePool), type(uint256).max);	// @audit-issue

58:        stEth.approve(address(wstEth), type(uint256).max);	// @audit-issue

81:        uint256 bal = ebtcToken.balanceOf(address(this));	// @audit-issue

84:            ebtcToken.transfer(msg.sender, bal);	// @audit-issue

93:        uint256 bal = stEth.sharesOf(address(this));	// @audit-issue

96:            stEth.transferShares(msg.sender, bal);	// @audit-issue

143:            coll + stEth.getSharesByPooledEth(_cdp._stEthBalanceIncrease) : 	// @audit-issue

144:            coll - stEth.getSharesByPooledEth(_cdp._stEthBalanceDecrease);	// @audit-issue

188:                stETH.getSharesByPooledEth(_cdp.stETHToDeposit),	// @audit-issue

288:            IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData.eBTCToMint * zapFeeBPS / BPS);	// @audit-issue

290:            super._openCdpForCallback(data);	// @audit-issue

309:                IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData._EBTCChange * zapFeeBPS / BPS);	// @audit-issue

312:            super._adjustCdpCallback(data);	// @audit-issue
```
[55](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L55-L55), [56](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L56-L56), [57](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L57-L57), [58](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L58-L58), [81](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L81-L81), [84](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L84-L84), [93](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L93-L93), [96](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L96-L96), [143](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L143-L143), [144](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L144-L144), [188](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L188-L188), [288](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L288-L288), [290](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L290-L290), [309](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L309-L309), [312](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L312-L312), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

228:        cdpId = sortedCdps.toCdpId(msg.sender, block.number, sortedCdps.nextCdpNonce());	// @audit-issue

246:            borrowerOperations.renouncePositionManagerApproval(msg.sender);	// @audit-issue

284:        uint256 debt = ICdpManager(address(cdpManager)).getSyncedCdpDebt(_cdpId);	// @audit-issue

294:        uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));	// @audit-issue

300:        uint256 _zapStEthBalanceAfter = stEth.balanceOf(address(this));	// @audit-issue

304:            borrowerOperations.renouncePositionManagerApproval(msg.sender);	// @audit-issue

342:        uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));	// @audit-issue

360:        uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));	// @audit-issue

378:        uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));	// @audit-issue

396:        uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));	// @audit-issue

416:        (uint256 debt, uint256 coll) = ICdpManager(address(cdpManager)).getSyncedDebtAndCollShares(_cdpId);	// @audit-issue

453:        uint256 _zapStEthBalanceDiff = stEth.balanceOf(address(this)) - _zapStEthBalanceBefore;	// @audit-issue

456:            borrowerOperations.renouncePositionManagerApproval(msg.sender);	// @audit-issue
```
[228](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L228-L228), [246](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L246-L246), [284](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L284-L284), [294](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L294-L294), [300](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L300-L300), [304](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L304-L304), [342](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L342-L342), [360](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L360-L360), [378](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L378-L378), [396](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L396-L396), [416](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L416-L416), [453](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L453-L453), [456](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L456-L456), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

28:        MIN_CHANGE = IMinChangeGetter(_borrowerOperations).MIN_CHANGE();	// @audit-issue

36:        uint256 _balBefore = stEth.balanceOf(address(this));	// @audit-issue

39:        uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;	// @audit-issue

44:        uint256 _wETHBalBefore = wrappedEth.balanceOf(address(this));	// @audit-issue

46:        uint256 _wETHReiceived = wrappedEth.balanceOf(address(this)) - _wETHBalBefore;	// @audit-issue

49:        IWrappedETH(address(wrappedEth)).withdraw(_wETHReiceived);	// @audit-issue

65:        uint256 _stETHBalBefore = stEth.balanceOf(address(this));	// @audit-issue

66:        IWstETH(address(wstEth)).unwrap(_initialWstETH);	// @audit-issue

67:        uint256 _stETHReiceived = stEth.balanceOf(address(this)) - _stETHBalBefore;	// @audit-issue

80:            uint256 _wstETHVal = IWstETH(address(wstEth)).wrap(_stEthVal);	// @audit-issue

91:            wstEth.transfer(msg.sender, _wstETHVal);	// @audit-issue

103:            stEth.transfer(msg.sender, _stEthVal);	// @audit-issue

109:        uint256 _balBefore = stEth.balanceOf(address(this));	// @audit-issue

111:        uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;	// @audit-issue
```
[28](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L28-L28), [36](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L36-L36), [39](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L39-L39), [44](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L44-L44), [46](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L46-L46), [49](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L49-L49), [65](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L65-L65), [66](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L66-L66), [67](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L67-L67), [80](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L80-L80), [91](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L91-L91), [103](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L103-L103), [109](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L109-L109), [111](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L111-L111), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

137:                sortedCdps.nextCdpNonce()	// @audit-issue

151:                sortedCdps.nextCdpNonce()	// @audit-issue

209:            ICdpManagerData.Cdp memory cdpInfo = cdpManager.Cdps(expectedCdpId);	// @audit-issue

220:            ICdpManagerData.Cdp memory cdpInfo = cdpManager.Cdps(checkParams.cdpId);	// @audit-issue

232:            ICdpManagerData.Cdp memory cdpInfo = cdpManager.Cdps(checkParams.cdpId);	// @audit-issue

253:        uint256 ebtcBal = ebtcToken.balanceOf(address(this));	// @audit-issue

254:        uint256 collateralBal = stETH.sharesOf(address(this));	// @audit-issue

257:            ebtcToken.transfer(msg.sender, ebtcBal);	// @audit-issue

261:            stETH.transferShares(msg.sender, collateralBal);	// @audit-issue

270:        IERC20(token).safeTransfer(msg.sender, amount);	// @audit-issue

455:        IERC20(swapData.tokenForSwap).safeApprove(	// @audit-issue

477:        IERC20(swapData.tokenForSwap).safeApprove(swapData.addressForApprove, 0);	// @audit-issue

491:                    IERC20(swapChecks[i].tokenToCheck).balanceOf(address(this)) >	// @audit-issue

544:        borrowerOperations.closeCdp(flData._cdpId);	// @audit-issue

563:        borrowerOperations.claimSurplusCollShares();	// @audit-issue
```
[137](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L137-L137), [151](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L151-L151), [209](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L209-L209), [220](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L220-L220), [232](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L232-L232), [253](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L253-L253), [254](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L254-L254), [257](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L257-L257), [261](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L261-L261), [270](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L270-L270), [455](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L455-L455), [477](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L477-L477), [491](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L491-L491), [544](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L544-L544), [563](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L563-L563), 


#### Recommendation

Review your smart contracts to identify and remove `block.number` and `block.timestamp` from the parameters of emitted events. Instead of manually adding these fields, rely on the Ethereum blockchain's inherent inclusion of this information within the block context. Simplify your event definitions to include only the essential data specific to the event's purpose, excluding universally available block metadata.

### Use assembly to check for `address(0)`
In Solidity, it's a common practice to check whether an Ethereum address variable is set to the zero address (`address(0)`) to handle various scenarios, such as token transfers or contract interactions. Typically, this check is performed using a conditional statement like `if (addressVariable == address(0))`.

However, using this approach in high-frequency or gas-sensitive operations can lead to unnecessary gas costs. A more gas-efficient alternative is to use inline assembly to perform the zero address check, which can significantly reduce gas consumption, especially in loops or complex contract logic.

By utilizing inline assembly for this specific check, you can optimize gas usage and make your Solidity code more efficient.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

46:            require(params.zapFeeReceiver != address(0));	// @audit-issue
```
[46](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L46-L46), 


#### Recommendation

To optimize gas usage in your Solidity code, consider using inline assembly for checking `address(0)`. This approach can significantly reduce gas costs, especially in high-frequency or gas-sensitive operations, leading to more efficient contract execution.

### Use assembly to check for `0`
Using assembly to check for zero can save gas by allowing more direct access to the evm and reducing some of the overhead associated with high-level operations in solidity.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

45:        if (params.zapFeeBPS > 0) {	// @audit-issue

83:        if (bal > 0) {	// @audit-issue

95:        if (bal > 0) {	// @audit-issue

142:        uint256 newColl = _cdp._stEthBalanceIncrease > 0 ? 	// @audit-issue

277:        if (zapFeeBPS > 0) {	// @audit-issue

295:        if (zapFeeBPS > 0) {	// @audit-issue
```
[45](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L45-L45), [83](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L83-L83), [95](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L95-L95), [142](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L142-L142), [277](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L277-L277), [295](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L295-L295), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

222:        if (_positionManagerPermit.length > 0) {	// @audit-issue

245:        if (_positionManagerPermit.length > 0) {	// @audit-issue

289:        if (_positionManagerPermit.length > 0) {	// @audit-issue

303:        if (_positionManagerPermit.length > 0) {	// @audit-issue

316:            _stEthBalanceIncrease > 0 || _stEthBalanceDecrease > 0 || _debtChange > 0,	// @audit-issue

326:            _stEthMarginIncrease == 0 || _stEthMarginDecrease == 0,	// @audit-issue

343:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue

361:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue

379:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue

397:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue

418:        if (_positionManagerPermit.length > 0) {	// @audit-issue

424:        if (!_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue

429:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue

455:        if (_positionManagerPermit.length > 0) {	// @audit-issue

459:        if (_zapStEthBalanceDiff > 0) {	// @audit-issue
```
[222](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L222-L222), [245](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L245-L245), [289](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L289-L289), [303](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L303-L303), [316](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L316-L316), [326](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L326-L326), [343](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L343-L343), [361](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L361-L361), [379](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L379-L379), [397](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L397-L397), [418](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L418-L418), [424](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L424-L424), [429](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L429-L429), [455](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L455-L455), [459](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L459-L459), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

137:            _change == 0 || _change >= MIN_CHANGE,	// @audit-issue
```
[137](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L137-L137), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

176:        if (operation.amountToTransferIn > 0) {	// @audit-issue

256:        if (ebtcBal > 0) {	// @audit-issue

260:        if (collateralBal > 0) {	// @audit-issue

329:        if (beforeSwapsLength > 0) {	// @audit-issue

347:        if (afterSwapsLength > 0) {	// @audit-issue
```
[176](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L176-L176), [256](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L256-L256), [260](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L260-L260), [329](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L329-L329), [347](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L347-L347), 


#### Recommendation

To optimize gas usage in your Solidity code, consider using inline assembly for checking `0`. This approach can significantly reduce gas costs, especially in high-frequency or gas-sensitive operations, leading to more efficient contract execution.

### Use assembly to write `address` storage values
Using assembly `{ sstore(state.slot, addr)}` instead of `state = addr` can save gas.


```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

49:        theOwner = params.owner;	// @audit-issue

50:        DEX = params.dex;	// @audit-issue

51:        zapFeeBPS = params.zapFeeBPS;	// @audit-issue

52:        zapFeeReceiver = params.zapFeeReceiver;	// @audit-issue
```
[49](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L49-L49), [50](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L50-L50), [51](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L51-L51), [52](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L52-L52), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

29:        wstEth = _wstEth;	// @audit-issue

30:        wrappedEth = _wEth;	// @audit-issue

31:        stEth = _stEth;	// @audit-issue
```
[29](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L29-L29), [30](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L30-L30), [31](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L31-L31), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

60:        borrowerOperations = IBorrowerOperations(_borrowerOperationsAddress);	// @audit-issue

61:        activePool = IActivePool(_activePool);	// @audit-issue

62:        cdpManager = ICdpCdps(_cdpManager);	// @audit-issue

63:        ebtcToken = IEBTCToken(_ebtc);	// @audit-issue

64:        stETH = ICollateralToken(_coll);	// @audit-issue

65:        sortedCdps = ISortedCdps(_sortedCdps);	// @audit-issue
```
[60](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L60-L60), [61](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L61-L61), [62](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L62-L62), [63](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L63-L63), [64](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L64-L64), [65](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L65-L65), 


#### Recommendation

To reduce gas costs in your Solidity code, consider using assembly with `{ sstore(state.slot, addr) }` for writing `address` storage values instead of `state = addr`. This approach can result in significant gas savings.

### Use assembly to emit an `event`
To efficiently emit events, it's possible to utilize assembly by making use of scratch space and the free memory pointer. This approach has the advantage of potentially avoiding the costs associated with memory expansion.

However, it's important to note that in order to safely optimize this process, it is preferable to cache and restore the free memory pointer.

A good example of such practice can be seen in [Solady's](https://github.com/Vectorized/solady/blob/main/src/tokens/ERC1155.sol#L167) codebase.


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

57:        emit ZapOperationEthVariant(	// @audit-issue

100:        emit ZapOperationEthVariant(	// @audit-issue

151:        emit ZapOperationEthVariant(	// @audit-issue

194:        emit ZapOperationEthVariant(	// @audit-issue
```
[57](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L57-L57), [100](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L100-L100), [151](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L151-L151), [194](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L194-L194), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

81:            emit ZapOperationEthVariant(	// @audit-issue

94:            emit ZapOperationEthVariant(	// @audit-issue
```
[81](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L81-L81), [94](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L94-L94), 


#### Recommendation

To optimize event emission in your Solidity code, consider using assembly with scratch space and the free memory pointer. This approach can help reduce gas costs by avoiding memory expansion expenses. However, it's crucial to ensure safe optimization by caching and restoring the free memory pointer, as demonstrated in examples like Solady's codebase.

### Use assembly to validate `msg.sender`
We can use assembly to efficiently validate `msg.sender` with the least amount of opcodes necessary. For more details check the following report [Here](https://code4rena.com/reports/2023-05-juicebox#g-06-use-assembly-to-validate-msgsender)


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

114:            msg.sender == address(wrappedEth),	// @audit-issue

282:        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for close!");	// @audit-issue

410:        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for adjust!");	// @audit-issue
```
[114](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L114-L114), [282](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L282-L282), [410](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L410-L410), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

45:        require(owner() == msg.sender, "Must be owner");	// @audit-issue

407:                msg.sender == address(borrowerOperations),	// @audit-issue

413:                msg.sender == address(activePool),	// @audit-issue
```
[45](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L45-L45), [407](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L407-L407), [413](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L413-L413), 


#### Recommendation

To optimize the validation of `msg.sender` in your Solidity code, consider using assembly to achieve this with the minimum number of opcodes required. You can refer to the detailed report [Here](https://code4rena.com/reports/2023-05-juicebox#g-06-use-assembly-to-validate-msgsender) for more insights and examples on efficient implementation.