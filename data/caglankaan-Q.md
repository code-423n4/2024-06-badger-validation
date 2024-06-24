### `block.number` means different things on different L2s
On Optimism, `block.number` is the L2 block number, but on Arbitrum, it's the L1 block number, and `ArbSys(address(100)).arbBlockNumber()` must be used. Furthermore, L2 block numbers often occur much more frequently than L1 block numbers (any may even occur on a per-transaction basis), so using block numbers for timing results in inconsistencies, especially when voting is involved across multiple chains. As of version 4.9, OpenZeppelin has [modified](https://blog.openzeppelin.com/introducing-openzeppelin-contracts-v4.9#governor) their governor code to use a clock rather than block numbers, to avoid these sorts of issues, but this still requires that the project [implement](https://docs.openzeppelin.com/contracts/4.x/governance#token_2) a [clock](https://eips.ethereum.org/EIPS/eip-6372) for each L2.

```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

228:        cdpId = sortedCdps.toCdpId(msg.sender, block.number, sortedCdps.nextCdpNonce());	// @audit-issue
```
[228](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L228-L228), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

136:                block.number,	// @audit-issue

150:                block.number,	// @audit-issue
```
[136](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L136-L136), [150](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L150-L150), 


#### Recommendation

Adopt a consistent timekeeping mechanism across L1 and L2 solutions when using `block.number` or similar timing mechanisms in your Solidity contracts. Consider using a standard clock or timestamp-based system, especially for functionalities like governance that require consistent timing across chains. For contracts on specific L2 solutions, ensure you're using the correct method to obtain block numbers consistent with the intended behavior. Keep abreast of standards and best practices, such as those suggested by OpenZeppelin, and implement appropriate solutions like EIP-6372 to provide a consistent and reliable timing mechanism across different blockchain environments.

### Return values of `transfer()`/`transferFrom()` not checked
Not all `ERC20` implementations `revert()` when there's a failure in `transfer()` or `transferFrom()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually transfer anything.

```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

45:        wrappedEth.transferFrom(msg.sender, address(this), _initialWETH);	// @audit-issue

61:            wstEth.transferFrom(msg.sender, address(this), _initialWstETH),	// @audit-issue

91:            wstEth.transfer(msg.sender, _wstETHVal);	// @audit-issue

110:        stEth.transferFrom(msg.sender, address(this), _initialStETH);	// @audit-issue
```
[45](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L45-L45), [61](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L61-L61), [91](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L91-L91), [110](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L110-L110), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

257:            ebtcToken.transfer(msg.sender, ebtcBal);	// @audit-issue
```
[257](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L257-L257), 


#### Recommendation

To ensure the reliability and security of token transfers in your smart contract, it's crucial to check the return values of the `transfer()` and `transferFrom()` functions. These functions often return a boolean value indicating the success or failure of the transfer operation. By checking this return value, you can accurately determine whether the transfer was successful and handle any potential errors or failures accordingly. Failing to check the return value may lead to unintended and unhandled transfer failures, which could have security and usability implications.

### Missing checks for `address(0)` in constructor/initializers
In Solidity, the Ethereum address `0x0000000000000000000000000000000000000000` is known as the "zero address". This address has significance because it's the default value for uninitialized address variables and is often used to represent an invalid or non-existent address. The "
Missing zero address control" issue arises when a Solidity smart contract does not properly check or prevent interactions with the zero address, leading to unintended behavior.
For instance, a contract might allow tokens to be sent to the zero address without any checks, which essentially burns those tokens as they become irretrievable. While sometimes this is intentional, without proper control or checks, accidental transfers could occur.    
        

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

28:        MIN_CHANGE = IMinChangeGetter(_borrowerOperations).MIN_CHANGE();	// @audit-issue

29:        wstEth = _wstEth;	// @audit-issue

30:        wrappedEth = _wEth;	// @audit-issue

31:        stEth = _stEth;	// @audit-issue
```
[28](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L28-L28), [29](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L29-L29), [30](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L30-L30), [31](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L31-L31), 


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

It is strongly recommended to implement checks to prevent the zero address from being set during the initialization of contracts. This can be achieved by adding require statements that ensure address parameters are not the zero address. 

### The `constructor`/`initialize` function lacks parameter validation.
Constructors and initialization functions play a critical role in contracts by setting important initial states when the contract is first deployed before the system starts. The parameters passed to the constructor and initialization functions directly affect the behavior of the contract / protocol. If incorrect parameters are provided, the system may fail to run, behave abnormally, be unstable, or lack security. Therefore, it's crucial to carefully check each parameter in the constructor and initialization functions. If an exception is found, the transaction should be rolled back.

```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

29:        wstEth = _wstEth;	// @audit-issue

30:        wrappedEth = _wEth;	// @audit-issue

31:        stEth = _stEth;	// @audit-issue
```
[29](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L29-L29), [30](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L30-L30), [31](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L31-L31), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

67:        willSweep = _sweepToCaller;	// @audit-issue
```
[67](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L67-L67), 


#### Recommendation

Incorporate comprehensive parameter validation in your contract's constructor and initialization functions. This should include checks for value ranges, address validity, and any other condition that defines a parameter as valid. Use `require` statements for validation, providing clear error messages for each condition. If any validation fails, the `require` statement will revert the transaction, preventing the contract from being deployed or initialized with invalid parameters.
Example:
```solidity
contract MyContract {
    address public owner;
    uint256 public someValue;

    constructor(address _owner, uint256 _someValue) {
        require(_owner != address(0), "Owner cannot be the zero address");
        require(_someValue > 0 && _someValue < 100, "SomeValue must be between 1 and 99");

        owner = _owner;
        someValue = _someValue;
    }
}

// For contracts using the proxy pattern and requiring initialization functions:
function initialize(address _owner, uint256 _someValue) public {
    require(_owner != address(0), "Owner cannot be the zero address");
    require(_someValue > 0 && _someValue < 100, "SomeValue must be between 1 and 99");

    owner = _owner;
    someValue = _someValue;
}
```


### Unsafe downcast may overflow
When a type is downcast to a smaller type, the higher order bits are discarded, resulting in the application of a modulo operation to the original value.

If the downcasted value is large enough, this may result in an overflow that will not revert.


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

151:        return address(uint160(_tmp));	// @audit-issue: Variable `_tmp` is type `uint256` and it is downcasted to `uint160`
```
[151](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L151-L151), 


#### Recommendation

When downcasting numbers to `addresses` in Solidity, be cautious of potential collisions. Downcasting truncates the upper bytes of the number, which can lead to multiple values mapping to the same `address`. To avoid collisions, consider using a more robust mapping strategy or additional checks to ensure unique mappings.

### `safeApprove()` is deprecated
The usage of deprecated library functions should be discouraged, as safeApprove is also potentially [dangerous](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/2219).

```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

455:        IERC20(swapData.tokenForSwap).safeApprove(	// @audit-issue

477:        IERC20(swapData.tokenForSwap).safeApprove(swapData.addressForApprove, 0);	// @audit-issue
```
[455](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L455-L455), [477](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L477-L477), 


#### Recommendation

Given its deprecation and associated risks, replace `safeApprove()` with the standard `approve()` method, incorporating best practices to manage allowances safely:
1. Before increasing an allowance, reduce it to zero to mitigate the risk of double-spending in transactions that are mined in quick succession.
2. Use events or transaction return values to verify the success of an `approve()` call.
3. Stay informed about ERC20 token standards and security practices to ensure your contracts interact with tokens safely and efficiently.


### The Contract Should `approve(0)` First
Some tokens (like USDT L199) do not work when changing the allowance from an existing non-zero allowance value. They must first be approved by zero and then the actual allowance must be approved.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

58:        stEth.approve(address(wstEth), type(uint256).max);	// @audit-issue
```
[58](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L58-L58), 


#### Recommendation

When developing functions that update allowances for ERC20 tokens, especially when interacting with tokens known to require this pattern, implement a two-step approval process. This process first sets the allowance for a spender to zero, and then sets it to the desired new value. For example:
```solidity
function resetAndApprove(address token, address spender, uint256 amount) public {
    IERC20(token).approve(spender, 0);
    IERC20(token).approve(spender, amount);
}
```


### Functions calling contracts/addresses with transfer hooks are missing reentrancy guards
Even if the function follows the best practice of check-effects-interaction, not using a reentrancy guard when there may be transfer hooks will open the users of this protocol up to [read-only reentrancies](https://chainsecurity.com/curve-lp-oracle-manipulation-post-mortem/) with no way to protect against it, except by block-listing the whole protocol.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

288:            IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData.eBTCToMint * zapFeeBPS / BPS);	// @audit-issue

309:                IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData._EBTCChange * zapFeeBPS / BPS);	// @audit-issue
```
[288](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L288-L288), [309](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L309-L309), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

45:        wrappedEth.transferFrom(msg.sender, address(this), _initialWETH);	// @audit-issue

61:            wstEth.transferFrom(msg.sender, address(this), _initialWstETH),	// @audit-issue

91:            wstEth.transfer(msg.sender, _wstETHVal);	// @audit-issue

103:            stEth.transfer(msg.sender, _stEthVal);	// @audit-issue

110:        stEth.transferFrom(msg.sender, address(this), _initialStETH);	// @audit-issue
```
[45](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L45-L45), [61](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L61-L61), [91](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L91-L91), [103](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L103-L103), [110](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L110-L110), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

177:            IERC20(operation.tokenToTransferIn).safeTransferFrom(	// @audit-issue

257:            ebtcToken.transfer(msg.sender, ebtcBal);	// @audit-issue

270:        IERC20(token).safeTransfer(msg.sender, amount);	// @audit-issue
```
[177](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L177-L177), [257](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L257-L257), [270](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L270-L270), 


#### Recommendation

To ensure the security of your protocol and protect users from potential vulnerabilities, consider implementing reentrancy guards in functions that call contracts or addresses with transfer hooks. Even if your function follows the check-effects-interaction pattern, using reentrancy guards is essential to prevent read-only reentrancy attacks, as demonstrated in incidents like the [Curve LP Oracle Manipulation](https://chainsecurity.com/curve-lp-oracle-manipulation-post-mortem/). Implementing reentrancy guards adds an extra layer of protection and safeguards your protocol against such attacks.

### Consider using OpenZeppelin’s `SafeCast` library to prevent unexpected overflows when casting from various type int/uint values
In Solidity, when casting from `int` to `uint` or vice versa, there is a risk of unexpected overflow or underflow, especially when dealing with large values. To mitigate this risk and ensure safe type conversions, you can consider using OpenZeppelin’s `SafeCast` library. This library provides functions that check for overflow/underflow conditions before performing the cast, helping you prevent potential issues related to type conversion in your smart contracts.

```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

150:        uint256 _tmp = uint256(cdpId) >> 96;	// @audit-issue

151:        return address(uint160(_tmp));	// @audit-issue
```
[150](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L150-L150), [151](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L151-L151), 


#### Recommendation

To enhance the safety and reliability of your Solidity smart contracts, it is advisable to utilize OpenZeppelin’s `SafeCast` library when casting between `int` and `uint` types. Incorporating this library into your codebase will help prevent unexpected overflows and underflows during type conversion, reducing the risk of vulnerabilities and ensuring secure contract execution.

### Numbers downcast to `addresses` may result in collisions
If a number is downcast to an `address` the upper bytes are truncated, which may mean that more than one value will map to the `address`


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

151:        return address(uint160(_tmp));	// @audit-issue
```
[151](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L151-L151), 


#### Recommendation

When downcasting numbers to `addresses` in Solidity, be cautious of potential collisions. Downcasting truncates the upper bytes of the number, which can lead to multiple values mapping to the same `address`. To avoid collisions, consider using a more robust mapping strategy or additional checks to ensure unique mappings.

### Revert on transfer to the zero address
It's good practice to revert a token transfer transaction if the recipient's address is the zero address. This can prevent unintentional transfers to the zero address due to accidental operations or programming errors. Many token contracts implement such a safeguard, such as [OpenZeppelin - ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/9e3f4d60c581010c4a3979480e07cc7752f124cc/contracts/token/ERC20/ERC20.sol#L232), [OpenZeppelin - ERC721](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/9e3f4d60c581010c4a3979480e07cc7752f124cc/contracts/token/ERC721/ERC721.sol#L142).

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

288:            IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData.eBTCToMint * zapFeeBPS / BPS);	// @audit-issue

309:                IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData._EBTCChange * zapFeeBPS / BPS);	// @audit-issue
```
[288](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L288-L288), [309](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L309-L309), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

91:            wstEth.transfer(msg.sender, _wstETHVal);	// @audit-issue

103:            stEth.transfer(msg.sender, _stEthVal);	// @audit-issue
```
[91](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L91-L91), [103](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L103-L103), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

257:            ebtcToken.transfer(msg.sender, ebtcBal);	// @audit-issue

270:        IERC20(token).safeTransfer(msg.sender, amount);	// @audit-issue
```
[257](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L257-L257), [270](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L270-L270), 


#### Recommendation

To enhance the security and reliability of your token contracts, it's advisable to implement a safeguard that reverts token transfer transactions if the recipient's address is the zero address. This practice helps prevent unintentional transfers to the zero address, reducing the risk of fund loss due to accidental operations or programming errors. Many token contracts, including [OpenZeppelin's ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/9e3f4d60c581010c4a3979480e07cc7752f124cc/contracts/token/ERC20/ERC20.sol#L232) and [ERC721](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/9e3f4d60c581010c4a3979480e07cc7752f124cc/contracts/token/ERC721/ERC721.sol#L142), incorporate this safeguard for added security.

### Missing contract existence checks before low-level calls
Low-level calls return success if there is no code present at the specified address. In addition to the zero-address checks, add a check to verify that `<address>.code.length > 0`.

```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

38:        payable(address(stEth)).call{value: _initialETH}("");	// @audit-issue
```
[38](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L38-L38), 


#### Recommendation

To enhance the security and reliability of your Solidity smart contracts, always include contract existence checks before making low-level calls. In addition to verifying that the address is not the zero address, also confirm that `<address>.code.length > 0`. These checks help ensure that the target address corresponds to a valid and functioning contract, reducing the risk of unexpected behavior and vulnerabilities in your contract interactions.

### External calls in an unbounded loop can result in a DoS
Consider limiting the number of iterations in loops that make external calls, as just a single one of them failing will result in a revert.

```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

491:                    IERC20(swapChecks[i].tokenToCheck).balanceOf(address(this)) >	// @audit-issue

492:                        swapChecks[i].expectedMinOut,	// @audit-issue
```
[491](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L491-L491), [492](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L492-L492), 


#### Recommendation

To mitigate the risk of Denial-of-Service (DoS) attacks in your Solidity code, it's important to limit the number of iterations in loops that involve external calls. A single failed external call in an unbounded loop can lead to a revert, causing disruptions in contract execution. Consider implementing safeguards, such as setting a maximum loop iteration count or employing strategies like batch processing, to reduce the impact of potential external call failures.

### Solidity version 0.8.20 might not work on all chains due to `PUSH0`
The Solidity version 0.8.20 employs the recently introduced PUSH0 opcode in the Shanghai EVM. This opcode might not be universally supported across all blockchain networks and Layer 2 solutions. Thus, as a result, it might be not possible to deploy solution with version 0.8.20 >= on some blockchains.

```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

2:pragma solidity ^0.8.17;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L2-L2), 


#### Recommendation

The Solidity version 0.8.20 employs the recently introduced PUSH0 opcode in the Shanghai EVM. This opcode might not be universally supported across all blockchain networks and Layer 2 solutions. Thus, as a result, it might be not possible to deploy solution with version 0.8.20 >= on some blockchains.

It is recommended to verify whether solution can be deployed on particular blockchain with the Solidity version 0.8.20 >=. Whenever such deployment is not possible due to lack of PUSH0 opcode support and lowering the Solidity version is a must, it is strongly advised to review all feature changes and bugfixes in [Solidity releases](https://soliditylang.org/blog/category/releases/). Some changes may have impact on current implementation and may impose a necessity of maintaining another version of solution.

### Use `increaseAllowance()`/`decreaseAllowance()` instead of `approve()`/`safeApprove()`
Changing an allowance with `approve()` brings the risk that someone may use both the old and the new allowance by unfortunate transaction ordering. [Refer to ERC20 API: An Attack Vector on the Approve/TransferFrom Methods](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit#heading=h.m9fhqynw2xvt). It is recommended to use the `increaseAllowance()`/`decreaseAllowance()` to avoid ths problem.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

58:        stEth.approve(address(wstEth), type(uint256).max);	// @audit-issue
```
[58](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L58-L58), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

455:        IERC20(swapData.tokenForSwap).safeApprove(	// @audit-issue

477:        IERC20(swapData.tokenForSwap).safeApprove(swapData.addressForApprove, 0);	// @audit-issue
```
[455](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L455-L455), [477](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L477-L477), 


#### Recommendation

To enhance the security of your Solidity smart contracts and avoid potential issues related to transaction ordering, it is advisable to use the `increaseAllowance()` and `decreaseAllowance()` functions instead of `approve()` or `safeApprove()`. These functions provide a safer and more atomic way to modify allowances and mitigate the risk associated with potential attack vectors like those described in the [ERC20 API: An Attack Vector on the Approve/TransferFrom Methods](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit#heading=h.m9fhqynw2xvt) document.

### Contracts are designed to receive ETH but do not implement function for withdrawal
The following contracts can receive ETH but do not provide a function for withdrawal. This means that any ETH sent to these contracts will be permanently stuck, unable to be retrieved by the contract owner or any other party. Additionally, this issue can also apply to baseTokens, resulting in locked tokens and potential loss of funds.


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

34:    function openCdpWithEth(	// @audit-issue
```
[34](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L34-L34), 


#### Recommendation

To prevent ETH and token lock-up and potential loss of funds, ensure that contracts designed to receive ETH or tokens implement a function for withdrawal. This function should allow contract owners and users to retrieve their funds when needed. Failure to provide a withdrawal mechanism can lead to locked assets and permanent loss, posing a significant risk to contract users and owners.

### Some tokens do not consider `type(uint256).max` as an infinite approval
Some tokens such as [COMP](https://github.com/compound-finance/compound-protocol/blob/a3214f67b73310d547e00fc578e8355911c9d376/contracts/Governance/Comp.sol#L89-L91) downcast such approvals to uint96 and use that as a raw value rather than interpreting it as an infinite approval. Eventually these approvals will reach zero, at which point the calling contract will no longer function properly.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

58:        stEth.approve(address(wstEth), type(uint256).max);	// @audit-issue
```
[58](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L58-L58), 


#### Recommendation

Ensure token contracts correctly interpret `type(uint256).max` as representing infinite approval. Avoid downcasting this value to smaller types, which could unintentionally impose a finite limit on approvals. If implementing a token contract, either respect `type(uint256).max` as infinite or clearly document and communicate any deviations from this norm. For contracts interacting with tokens, be aware of how different tokens handle this value and plan for potential edge cases where infinite approvals are treated as finite.

### Contracts use infinite approvals with no means to revoke
Infinite approvals on external contracts can be dangerous if the target becomes compromised. See [here](https://revoke.cash/exploits) for a list of approval exploits. The following contracts are vulnerable to such attacks since they have no functionality to revoke the approval (call `approve` with amount `0`). Consider enabling the contract to revoke approval in emergency situations.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

58:        stEth.approve(address(wstEth), type(uint256).max);	// @audit-issue
```
[58](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L58-L58), 


#### Recommendation

Implement functionalities in your Solidity contracts to revoke infinite approvals. Allow users or designated entities (like contract administrators) to reset approval amounts to zero, providing a safeguard against compromised external contracts. This can be a dedicated function or part of a broader emergency response mechanism. Clearly document and communicate this feature to users for transparency and ease of use. Additionally, consider monitoring and auditing approved contracts regularly to identify and respond to potential vulnerabilities promptly.

### Return values of `approve` not checked.
Not all `IERC20` implementations `revert` when there's a failure in `approve`. The function signature has a boolean return value and they indicate errors that way instead.

By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything. 


```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

58:        stEth.approve(address(wstEth), type(uint256).max);	// @audit-issue
```
[58](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L58-L58), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

455:        IERC20(swapData.tokenForSwap).safeApprove(	// @audit-issue

477:        IERC20(swapData.tokenForSwap).safeApprove(swapData.addressForApprove, 0);	// @audit-issue
```
[455](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L455-L455), [477](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L477-L477), 


#### Recommendation

To ensure the accuracy of token approvals and prevent potential issues, it's important to check the return values when calling the `approve` function of an `IERC20` contract. While some implementations use a boolean return value to indicate errors, not checking this return value may allow operations to proceed without the intended approval. Always verify the return value to confirm the success of the approval operation.

### Large transfers may not work with some `ERC20` tokens
Not all IERC20 implementations are totally compliant, and some (e.g UNI, COMP) may fail if the valued passed is larger than uint96. [Source](https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers)


```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

288:            IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData.eBTCToMint * zapFeeBPS / BPS);	// @audit-issue

309:                IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData._EBTCChange * zapFeeBPS / BPS);	// @audit-issue
```
[288](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L288-L288), [309](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L309-L309), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

45:        wrappedEth.transferFrom(msg.sender, address(this), _initialWETH);	// @audit-issue

61:            wstEth.transferFrom(msg.sender, address(this), _initialWstETH),	// @audit-issue

91:            wstEth.transfer(msg.sender, _wstETHVal);	// @audit-issue

103:            stEth.transfer(msg.sender, _stEthVal);	// @audit-issue

110:        stEth.transferFrom(msg.sender, address(this), _initialStETH);	// @audit-issue
```
[45](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L45-L45), [61](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L61-L61), [91](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L91-L91), [103](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L103-L103), [110](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L110-L110), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

177:            IERC20(operation.tokenToTransferIn).safeTransferFrom(	// @audit-issue

257:            ebtcToken.transfer(msg.sender, ebtcBal);	// @audit-issue

270:        IERC20(token).safeTransfer(msg.sender, amount);	// @audit-issue
```
[177](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L177-L177), [257](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L257-L257), [270](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L270-L270), 


#### Recommendation

When transferring tokens using ERC-20 contracts, such as UNI or COMP, it's important to be aware that not all IERC20 implementations are fully compliant, and they may not support large transfer values exceeding uint96. To avoid potential issues, it's essential to check the documentation or source code of the specific token you're using and ensure that your transfers adhere to the token's limitations.

### Large approvals may not work with some `ERC20` tokens
Not all IERC20 implementations are totally compliant, and some (e.g UNI, COMP) may fail if the valued passed is larger than uint96. [Source](https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers)

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

58:        stEth.approve(address(wstEth), type(uint256).max);	// @audit-issue
```
[58](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L58-L58), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

455:        IERC20(swapData.tokenForSwap).safeApprove(	// @audit-issue

477:        IERC20(swapData.tokenForSwap).safeApprove(swapData.addressForApprove, 0);	// @audit-issue
```
[455](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L455-L455), [477](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L477-L477), 


#### Recommendation

When working with ERC-20 tokens, particularly tokens like UNI or COMP, be aware that not all IERC20 implementations are fully compliant, and they may not support large approval values exceeding uint96. To avoid potential issues, it's essential to check the documentation or source code of the specific token you're interacting with and ensure that your approvals conform to the token's limitations.

### Unsafe ERC20 operation `approve()`
Approve call do not handle non-standard erc20 behavior. Use `safeApprove` instead of `approve`.
- Some token contracts do not return any value.
- Some token contracts revert the transaction when the allowance is not zero.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

58:        stEth.approve(address(wstEth), type(uint256).max);	// @audit-issue
```
[58](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L58-L58), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

455:        IERC20(swapData.tokenForSwap).safeApprove(	// @audit-issue

477:        IERC20(swapData.tokenForSwap).safeApprove(swapData.addressForApprove, 0);	// @audit-issue
```
[455](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L455-L455), [477](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L477-L477), 


#### Recommendation

Use `safeApprove` instead of `approve`

### Possible loss of precision
Division by large numbers may result in precision loss due to rounding down, or even the result being erroneously equal to zero. Consider adding checks on the numerator to ensure precision loss is handled appropriately.


```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

268:                    value:  _totalCollateral * _collValidationBuffer / BPS,	// @audit-issue

288:            IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData.eBTCToMint * zapFeeBPS / BPS);	// @audit-issue

309:                IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData._EBTCChange * zapFeeBPS / BPS);	// @audit-issue
```
[268](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L268-L268), [288](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L288-L288), [309](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L309-L309), 


#### Recommendation

Incorporate strategies in your Solidity contracts to mitigate precision loss in division operations. This can include:
1. Performing checks on the numerator and denominator to ensure they are within a reasonable range to avoid significant rounding errors.
2. Considering the use of fixed-point arithmetic libraries or scaling factors to handle divisions with higher precision.
3. Clearly documenting any inherent limitations of your division logic and providing guidelines for inputs to minimize unexpected behavior.
Always thoroughly test division operations under various scenarios to ensure that the outcomes are consistent with your contract's intended logic and accuracy requirements.

### `approve` can revert if the current approval is not zero
Some tokens like USDT check for the current approval, and they revert if it's not zero. While Tether is known to do this, it applies to other tokens as well, which are trying to protect against [this](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit) attack vector.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

58:        stEth.approve(address(wstEth), type(uint256).max);	// @audit-issue
```
[58](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L58-L58), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

455:        IERC20(swapData.tokenForSwap).safeApprove(	// @audit-issue

477:        IERC20(swapData.tokenForSwap).safeApprove(swapData.addressForApprove, 0);	// @audit-issue
```
[455](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L455-L455), [477](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L477-L477), 


#### Recommendation

When working with tokens that may revert when calling the `approve` function, be cautious and ensure that the current approval is set to zero before attempting to set a new approval. Tokens like USDT and others may follow this behavior to protect against potential attack vectors. Always check the token's documentation and handle approvals accordingly to avoid unexpected reverts.

### `approve` will always revert as the `IERC20` interface mismatch
Some tokens, such as [USDT](https://etherscan.io/token/0xdac17f958d2ee523a2206206994597c13d831ec7#code#L199), have a different implementation for the approve function: when the address is cast to a compliant `IERC20` interface and the approve function is used, it will always revert due to the interface mismatch.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

58:        stEth.approve(address(wstEth), type(uint256).max);	// @audit-issue
```
[58](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L58-L58), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

455:        IERC20(swapData.tokenForSwap).safeApprove(	// @audit-issue

477:        IERC20(swapData.tokenForSwap).safeApprove(swapData.addressForApprove, 0);	// @audit-issue
```
[455](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L455-L455), [477](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L477-L477), 


#### Recommendation

When interacting with tokens like USDT that have a different implementation for the `approve` function, ensure that you use the correct interface and method to avoid reverts due to interface mismatches. Instead of casting to `IERC20`, use the specific interface that matches the token's implementation. Always review the token's documentation or contract to determine the correct interface and method to use when working with such tokens.

### Non-compliant `IERC20` tokens may revert with `transfer`
Some `IERC20` tokens (e.g. `BNB`, `OMG`, `USDT`) do not implement the standard properly, but they are still accepted by most code that accepts `ERC20` tokens.

For example, `USDT` transfer functions on L1 do not return booleans: when casted to `IERC20`, their function signatures do not match, and therefore the calls made will revert.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

288:            IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData.eBTCToMint * zapFeeBPS / BPS);	// @audit-issue

309:                IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData._EBTCChange * zapFeeBPS / BPS);	// @audit-issue
```
[288](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L288-L288), [309](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L309-L309), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

45:        wrappedEth.transferFrom(msg.sender, address(this), _initialWETH);	// @audit-issue

61:            wstEth.transferFrom(msg.sender, address(this), _initialWstETH),	// @audit-issue

91:            wstEth.transfer(msg.sender, _wstETHVal);	// @audit-issue

103:            stEth.transfer(msg.sender, _stEthVal);	// @audit-issue

110:        stEth.transferFrom(msg.sender, address(this), _initialStETH);	// @audit-issue
```
[45](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L45-L45), [61](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L61-L61), [91](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L91-L91), [103](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L103-L103), [110](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L110-L110), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

177:            IERC20(operation.tokenToTransferIn).safeTransferFrom(	// @audit-issue

257:            ebtcToken.transfer(msg.sender, ebtcBal);	// @audit-issue

270:        IERC20(token).safeTransfer(msg.sender, amount);	// @audit-issue
```
[177](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L177-L177), [257](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L257-L257), [270](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L270-L270), 


#### Recommendation

When dealing with tokens that may not fully comply with the `IERC20` standard, it's important to exercise caution and carefully verify the behavior of the specific token in question. In cases where tokens do not return booleans for transfer functions, consider implementing additional error-handling mechanisms to account for potential failures. This may include checking the token balance before and after the transfer to detect any discrepancies or using try-catch blocks to handle potential exceptions. Ensuring robust error handling can help your smart contract interact more gracefully with non-compliant tokens while maintaining security and reliability.

### Unsafe use of `transfer()`/`transferFrom()` with `IERC20`
Some tokens do not implement the `ERC20` standard properly but are still accepted by most code that accepts `ERC20` tokens. For example Tether (USDT)'s `transfer()` and `transferFrom()` functions on L1 do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to `IERC20`, their [function signatures](https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca) do not match and therefore the calls made, revert (see [this](https://gist.github.com/IllIllI000/2b00a32e8f0559e8f386ea4f1800abc5) link for a test case). Use OpenZeppelin’s SafeERC20's `safeTransfer()`/`safeTransferFrom()` instead

```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

45:        wrappedEth.transferFrom(msg.sender, address(this), _initialWETH);	// @audit-issue

61:            wstEth.transferFrom(msg.sender, address(this), _initialWstETH),	// @audit-issue

91:            wstEth.transfer(msg.sender, _wstETHVal);	// @audit-issue

103:            stEth.transfer(msg.sender, _stEthVal);	// @audit-issue

110:        stEth.transferFrom(msg.sender, address(this), _initialStETH);	// @audit-issue
```
[45](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L45-L45), [61](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L61-L61), [91](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L91-L91), [103](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L103-L103), [110](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L110-L110), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

257:            ebtcToken.transfer(msg.sender, ebtcBal);	// @audit-issue
```
[257](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L257-L257), 


#### Recommendation

When dealing with tokens that may not fully comply with the `IERC20` standard, it's important to exercise caution and carefully verify the behavior of the specific token in question. In cases where tokens do not return booleans for transfer functions, consider implementing additional error-handling mechanisms to account for potential failures. This may include checking the token balance before and after the transfer to detect any discrepancies or using try-catch blocks to handle potential exceptions. Ensuring robust error handling can help your smart contract interact more gracefully with non-compliant tokens while maintaining security and reliability.

### Floating Pragma
A "floating pragma" in Solidity refers to the practice of using a pragma statement that does not specify a fixed compiler version but instead allows the contract to be compiled with any compatible compiler version. This issue arises when pragma statements like `pragma solidity ^0.8.0;` are used without a specific version number, allowing the contract to be compiled with the latest available compiler version. This can lead to various compatibility and stability issues.


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

2:pragma solidity ^0.8.17;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L2-L2), 


#### Recommendation

Consider locking the pragma version whenever possible and avoid using a floating pragma in the final deployment. [Consider known bugs](https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.

### Use transfer libraries instead of low level calls
Consider using `SafeTransferLib.safeTransferETH` or `Address.sendValue` for clearer semantic meaning instead of using a low level call.

```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

38:        payable(address(stEth)).call{value: _initialETH}("");	// @audit-issue
```
[38](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L38-L38), 


#### Recommendation

To improve code readability and safety, consider using higher-level transfer libraries like `SafeTransferLib.safeTransferETH` or `Address.sendValue` instead of low-level calls for handling Ether transfers. These libraries provide clearer semantic meaning and help prevent common pitfalls associated with low-level calls.

### Unused arguments should be removed or implemented
In Solidity, functions often have arguments that are intended for specific operations or logic within the function. However, sometimes these arguments remain unused in the function body, either due to changes during development or oversight. Unused arguments in functions can lead to confusion, making the code less readable and potentially misleading for other developers who might use or audit the contract. Moreover, they can create a false impression of the function's purpose and behavior. It's crucial to either implement these arguments in the function's logic as originally intended or remove them to ensure clarity and efficiency of the code.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

66:        FlashLoanType flType,	// @audit-issue: flType

67:        uint256 borrowAmount,	// @audit-issue: borrowAmount

68:        LeverageMacroOperation calldata operation,	// @audit-issue: operation

69:        PostOperationCheck postCheckType,	// @audit-issue: postCheckType

70:        PostCheckParams calldata checkParams	// @audit-issue: checkParams
```
[66](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L66-L66), [67](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L67-L67), [68](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L68-L68), [69](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L69-L69), [70](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L70-L70), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

210:        uint256 _stEthMarginAmount,	// @audit-issue: _stEthMarginAmount
```
[210](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L210-L210), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

397:        uint256 amount,	// @audit-issue: amount

398:        uint256 fee,	// @audit-issue: fee
```
[397](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L397-L397), [398](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L398-L398), 


#### Recommendation

Eliminate unused arguments in Solidity function definitions to enhance code clarity and efficiency. If an argument is currently not utilized in the function's logic, assess its potential future utility. If it serves no purpose, removing it simplifies the function signature and aligns the code more closely with its actual operation, contributing to a cleaner and more understandable contract structure.

### Unused import
The identifier is imported but never used within the file.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

4:import {IPositionManagers} from "@ebtc/contracts/Interfaces/IBorrowerOperations.sol";	// @audit-issue: IPositionManagers not used in the contract.

5:import {IPriceFeed} from "@ebtc/contracts/Interfaces/IPriceFeed.sol";	// @audit-issue: IPriceFeed not used in the contract.
```
[4](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L4-L4), [5](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L5-L5), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

4:import {IERC3156FlashLender} from "@ebtc/contracts/Interfaces/IERC3156FlashLender.sol";	// @audit-issue: IERC3156FlashLender not used in the contract.

6:import {ICdpManagerData} from "@ebtc/contracts/Interfaces/ICdpManagerData.sol";	// @audit-issue: ICdpManagerData not used in the contract.

8:import {IBorrowerOperations} from "@ebtc/contracts/Interfaces/IBorrowerOperations.sol";	// @audit-issue: IBorrowerOperations not used in the contract.

9:import {IPositionManagers} from "@ebtc/contracts/Interfaces/IPositionManagers.sol";	// @audit-issue: IPositionManagers not used in the contract.

10:import {IERC20} from "@ebtc/contracts/Dependencies/IERC20.sol";	// @audit-issue: IERC20 not used in the contract.

12:import {EbtcBase} from "@ebtc/contracts/Dependencies/EbtcBase.sol";	// @audit-issue: EbtcBase not used in the contract.

13:import {IStETH} from "./interface/IStETH.sol";	// @audit-issue: IStETH not used in the contract.

14:import {IWrappedETH} from "./interface/IWrappedETH.sol";	// @audit-issue: IWrappedETH not used in the contract.

16:import {IWstETH} from "./interface/IWstETH.sol";	// @audit-issue: IWstETH not used in the contract.
```
[4](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L4-L4), [6](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L6-L6), [8](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L8-L8), [9](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L9-L9), [10](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L10-L10), [12](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L12-L12), [13](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L13-L13), [14](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L14-L14), [16](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L16-L16), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

4:import {IPriceFeed} from "@ebtc/contracts/Interfaces/IPriceFeed.sol";	// @audit-issue: IPriceFeed not used in the contract.

5:import {ICdpManagerData} from "@ebtc/contracts/Interfaces/ICdpManager.sol";	// @audit-issue: ICdpManagerData not used in the contract.
```
[4](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L4-L4), [5](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L5-L5), 


#### Recommendation

Regularly review your Solidity code to remove unused imports. This practice declutters the codebase, making it easier to understand and maintain. In addition, consider using tools or IDE features that can automatically detect and highlight unused imports for cleanup. Keeping imports limited to only what is necessary helps maintain a clear understanding of the contract's dependencies and reduces potential confusion for developers working on or reviewing the code.

### `public` functions not called by the contract should be declared `external` instead
In Solidity, function visibility is an important aspect that determines how and where a function can be called from. Two commonly used visibilities are `public` and `external`. A `public` function can be called both from other functions inside the same contract and from outside transactions, while an `external` function can only be called from outside the contract.
A potential pitfall in smart contract development is the misuse of the `public` keyword for functions that are only meant to be accessed externally. When a function is not used internally within a contract and is only intended for external calls, it should be labeled as `external` rather than `public`. Using `public` unnecessarily can introduce potential vulnerabilities and also make the contract consume more gas than required. This is because `public` functions have to add additional code to handle both internal and external calls, while `external` functions can be more optimized since they only handle external calls.


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

267:    function sweepToken(address token, uint256 amount) public {	// @audit-issue
```
[267](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L267-L267), 


#### Recommendation

To optimize gas usage and improve code clarity, declare functions that are not called internally within the contract and are intended for external access as `external` rather than `public`. This ensures that these functions are only callable externally, reducing unnecessary gas consumption and potential security risks.

### `if`-statement can be converted to a ternary
The code can be made more compact while also increasing readability by converting the following `if`-statements to ternaries (e.g. `foo += (x > y) ? a : b`)

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

45:        if (params.zapFeeBPS > 0) {	// @audit-issue
46:            require(params.zapFeeReceiver != address(0));
47:        }

83:        if (bal > 0) {	// @audit-issue
84:            ebtcToken.transfer(msg.sender, bal);
85:        }

95:        if (bal > 0) {	// @audit-issue
96:            stEth.transferShares(msg.sender, bal);
97:        }

113:        if (_cdp._isDebtIncrease) {	// @audit-issue
114:            // for debt increases, we flash borrow stETH
115:            // trade eBTC -> stETH for repayment
116:            op.swapsAfter = _getSwapOperations(
117:                address(ebtcToken), 
118:                address(stETH),
119:                _tradeData
120:            );
121:        } else {
122:            // for debt decreases (unwinding), we flash borrow eBTC
123:            // trade stETH -> eBTC for repayment
124:            op.swapsAfter = _getSwapOperations(
125:                address(stETH),
126:                address(ebtcToken),
127:                _tradeData
128:            );
129:        }

244:        if (_tradeData.performSwapChecks) {	// @audit-issue
245:            swaps[0].swapChecks = _getSwapChecks(_tokenOut, _tradeData.expectedMinOut);            
246:        }
```
[45](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L45-L47), [83](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L83-L85), [95](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L95-L97), [113](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L113-L129), [244](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L244-L246), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

245:        if (_positionManagerPermit.length > 0) {	// @audit-issue
246:            borrowerOperations.renouncePositionManagerApproval(msg.sender);
247:        }

303:        if (_positionManagerPermit.length > 0) {	// @audit-issue
304:            borrowerOperations.renouncePositionManagerApproval(msg.sender);
305:        }

343:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue
344:            _params.stEthMarginBalance = _convertRawEthToStETH(_params.stEthMarginBalance);
345:        }

361:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue
362:            _params.stEthMarginBalance = _convertWstEthToStETH(_params.stEthMarginBalance);
363:        }

379:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue
380:            _params.stEthMarginBalance = _convertWrappedEthToStETH(_params.stEthMarginBalance);
381:        }

397:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue
398:            _params.stEthMarginBalance = _transferInitialStETHFromCaller(_params.stEthMarginBalance);
399:        }

424:        if (!_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue
425:            marginDecrease += _params.stEthMarginBalance;
426:        }

429:        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {	// @audit-issue
430:            marginIncrease += _params.stEthMarginBalance;
431:        }

455:        if (_positionManagerPermit.length > 0) {	// @audit-issue
456:            borrowerOperations.renouncePositionManagerApproval(msg.sender);
457:        }

459:        if (_zapStEthBalanceDiff > 0) {	// @audit-issue
460:            _transferStEthToCaller(
461:                _cdpId,
462:                EthVariantZapOperationType.AdjustCdp,
463:                _params.useWstETHForDecrease,
464:                _zapStEthBalanceDiff
465:            );
466:        }
```
[245](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L245-L247), [303](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L303-L305), [343](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L343-L345), [361](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L361-L363), [379](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L379-L381), [397](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L397-L399), [424](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L424-L426), [429](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L429-L431), [455](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L455-L457), [459](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L459-L466), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

176:        if (operation.amountToTransferIn > 0) {	// @audit-issue
177:            IERC20(operation.tokenToTransferIn).safeTransferFrom(
178:                msg.sender,
179:                address(this),
180:                operation.amountToTransferIn
181:            );
182:        }

192:        } else if (flType == FlashLoanType.stETH) {	// @audit-issue
193:            IERC3156FlashLender(address(activePool)).flashLoan(
194:                IERC3156FlashBorrower(address(this)),
195:                address(stETH),
196:                borrowAmount,
197:                abi.encode(operation)
198:            );
199:        } else {
200:            // No leverage, just do the operation
201:            _handleOperation(operation);
202:        }

241:        if (willSweep) {	// @audit-issue
242:            sweepToCaller();
243:        }

256:        if (ebtcBal > 0) {	// @audit-issue
257:            ebtcToken.transfer(msg.sender, ebtcBal);
258:        }

260:        if (collateralBal > 0) {	// @audit-issue
261:            stETH.transferShares(msg.sender, collateralBal);
262:        }

285:        } else if (check.operator == Operator.equal) {	// @audit-issue
286:            require(check.value == valueToCheck, "!LeverageMacroReference: equal post check");
287:        } else {
288:            revert("Operator not found");
289:        }

329:        if (beforeSwapsLength > 0) {	// @audit-issue
330:            _doSwaps(operation.swapsBefore);
331:        }

342:        } else if (operation.operationType == OperationType.ClaimSurplusOperation) {	// @audit-issue
343:            _claimSurplusCallback();
344:        }

347:        if (afterSwapsLength > 0) {	// @audit-issue
348:            _doSwaps(operation.swapsAfter);
349:        }

405:        if (token == address(ebtcToken)) {	// @audit-issue
406:            require(
407:                msg.sender == address(borrowerOperations),
408:                "LeverageMacroReference: wrong lender for eBTC flashloan"
409:            );
410:        } else {
411:            // Enforce that this is either eBTC or stETH
412:            require(
413:                msg.sender == address(activePool),
414:                "LeverageMacroReference: wrong lender for stETH flashloan"
415:            );
416:        }
```
[176](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L176-L182), [192](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L192-L202), [241](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L241-L243), [256](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L256-L258), [260](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L260-L262), [285](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L285-L289), [329](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L329-L331), [342](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L342-L344), [347](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L347-L349), [405](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L405-L416), 


#### Recommendation

Consider using single line if statements as ternary.

### Missing Revert Messages in `require`/`revert` Statements
In Solidity, the `require` function is commonly used to enforce certain conditions or invariants in the code. If the condition inside `require` evaluates to `false`, the function will throw an exception and revert all changes made during the transaction. Along with this condition check, `require` can also accept a second argument — a string that provides a descriptive error message to convey the reason for the transaction's failure.

When `require` is used without this descriptive error message, as in `require(someBoolean)`, it results in less informative feedback to the users or interacting contracts. This lack of clarity can pose challenges in debugging failed transactions or understanding why an operation was not allowed.

On the other hand, a well-structured error message, like `require(someBoolean, "this is an error msg")`, offers insight into the failure's cause, making it much easier for developers, users, and auditors to identify and address issues.


```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

46:            require(params.zapFeeReceiver != address(0));	// @audit-issue
```
[46](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L46-L46), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

502:        require(addy != address(borrowerOperations));	// @audit-issue

503:        require(addy != address(sortedCdps));	// @audit-issue

504:        require(addy != address(activePool));	// @audit-issue

505:        require(addy != address(cdpManager));	// @audit-issue

506:        require(addy != address(this)); // If it could call this it could fake the forwarded caller	// @audit-issue
```
[502](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L502-L502), [503](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L503-L503), [504](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L504-L504), [505](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L505-L505), [506](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L506-L506), 


#### Recommendation

Consider adding an error message to require statements.

### Contract uses both `require()`/`revert()` as well as custom errors
Consider using just one method in a single file

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

15:abstract contract LeverageZapRouterBase is ZapRouterBase, LeverageMacroBase, ReentrancyGuard, IEbtcLeverageZapRouter {	// @audit-issue
```
[15](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L15-L15), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

26:abstract contract LeverageMacroBase {	// @audit-issue
```
[26](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L26-L26), 


#### Recommendation

Consistently use either `require()`/`revert()` or custom errors for error handling in your contract to maintain code consistency and readability. Using both methods in the same contract can lead to confusion and make the code harder to understand.

### Use a `modifier` instead of `require` to check for `msg.sender`
If some functions are only allowed to be called by some specific users, consider using a `modifier` instead of checking with a `require` statement, especially if this check is done in multiple functions.

```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

114:            msg.sender == address(wrappedEth),	// @audit-issue

282:        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for close!");	// @audit-issue

410:        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for adjust!");	// @audit-issue
```
[114](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L114-L114), [282](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L282-L282), [410](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L410-L410), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

61:            wstEth.transferFrom(msg.sender, address(this), _initialWstETH),	// @audit-issue
```
[61](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L61-L61), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

45:        require(owner() == msg.sender, "Must be owner");	// @audit-issue

407:                msg.sender == address(borrowerOperations),	// @audit-issue

413:                msg.sender == address(activePool),	// @audit-issue
```
[45](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L45-L45), [407](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L407-L407), [413](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L413-L413), 


#### Recommendation

To improve code readability and maintainability, consider using a `modifier` to check for `msg.sender` in functions that should only be called by specific users. This not only simplifies the code but also makes it more consistent and easier to audit.

### Consider moving `msg.sender` checks to `modifier`s
If some functions are only allowed to be called by some specific users, consider using a modifier instead of checking with a require statement, especially if this check is done in multiple functions.

```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

113:        require(	// @audit-issue
114:            msg.sender == address(wrappedEth),
115:            "EbtcLeverageZapRouter: only allow Wrapped ETH to send Ether!"
116:        );

282:        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for close!");	// @audit-issue

410:        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for adjust!");	// @audit-issue
```
[113](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L113-L116), [282](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L282-L282), [410](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L410-L410), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

60:        require(	// @audit-issue
61:            wstEth.transferFrom(msg.sender, address(this), _initialWstETH),
62:            "EbtcZapRouter: transfer wstETH failure!"
63:        );
```
[60](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L60-L63), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

45:        require(owner() == msg.sender, "Must be owner");	// @audit-issue

406:            require(	// @audit-issue
407:                msg.sender == address(borrowerOperations),
408:                "LeverageMacroReference: wrong lender for eBTC flashloan"
409:            );

412:            require(	// @audit-issue
413:                msg.sender == address(activePool),
414:                "LeverageMacroReference: wrong lender for stETH flashloan"
415:            );
```
[45](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L45-L45), [406](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L406-L409), [412](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L412-L415), 


#### Recommendation

Consider refactoring your code by moving `msg.sender` checks to modifiers when certain functions are only allowed to be called by specific users. This approach can enhance code readability, reduce redundancy, and make it easier to maintain access control logic.

### Constants in comparisons should appear on the left side
Doing so will prevent [typo bugs](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html)

```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

326:            _stEthMarginIncrease == 0 || _stEthMarginDecrease == 0,	// @audit-issue
```
[326](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L326-L326), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

137:            _change == 0 || _change >= MIN_CHANGE,	// @audit-issue
```
[137](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L137-L137), 


#### Recommendation

To prevent typo bugs and improve code readability, it's advisable to place constants on the left side of comparisons. This coding practice helps catch accidental assignment (=) instead of comparison (==) and enhances code quality.

### Convert duplicated codes to modifier/functions
Duplicated code in Solidity contracts, especially when repeated across multiple functions, can lead to inefficiencies and increased maintenance challenges. It not only bloats the contract but also makes it harder to implement changes consistently across the codebase. By identifying common patterns or checks that are repeated and abstracting them into modifiers or separate functions, the code can be made more concise, readable, and maintainable. This practice not only reduces the overall bytecode size, potentially lowering deployment and execution costs, but also enhances the contract's reliability by ensuring consistency in how these common checks or operations are executed.

```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

222:        if (_positionManagerPermit.length > 0) {	// @audit-issue: Same if statement in line(s): ['289', '418']

245:        if (_positionManagerPermit.length > 0) {	// @audit-issue: Same if statement in line(s): ['303', '455']
```
[222](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L222-L222), [245](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L245-L245), 


#### Recommendation

Identify and consolidate duplicated code blocks in Solidity contracts into reusable modifiers or functions. This approach streamlines the contract by eliminating redundancy, thereby improving clarity, maintainability, and potentially reducing gas costs. For common checks, consider using modifiers for concise and consistent enforcement of conditions. For reusable logic, encapsulate it in functions to avoid code duplication and simplify future updates or bug fixes.

### Style guide: Function ordering does not follow the Solidity style guide
According to the Solidity [style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions), functions should be laid out in the following order :`constructor()`, `receive()`, `fallback()`, `external`, `public`, `internal`, `private`, but the cases below do not follow this pattern

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

65:    function doOperation(	// @audit-issue
```
[65](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L65-L65), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

112:    receive() external payable {	// @audit-issue
```
[112](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L112-L112), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

51:    constructor(	// @audit-issue
```
[51](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L51-L51), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/latest/style-guide.html).

### Duplicate import statements
Contracts often depend on libraries and other contracts to modularize code and reuse functionalities. However, redundant imports occur when a contract imports a library or another contract that is already imported by one of its dependencies. This redundancy does not impact the compiled bytecode but can clutter the codebase, making it harder to understand the direct dependencies of each contract. Simplifying imports by removing these redundancies enhances code readability and maintainability.

```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

7:import {ICdpManager} from "@ebtc/contracts/Interfaces/ICdpManager.sol";	// @audit-issue: Same library is also imported on: `['LeverageZapRouterBase']`, at lines: `[6]`

8:import {IBorrowerOperations} from "@ebtc/contracts/Interfaces/IBorrowerOperations.sol";	// @audit-issue: Same library is also imported on: `['LeverageZapRouterBase']`, at lines: `[4]`

10:import {IERC20} from "@ebtc/contracts/Dependencies/IERC20.sol";	// @audit-issue: Same library is also imported on: `['LeverageZapRouterBase']`, at lines: `[13]`

11:import {SafeERC20} from "@ebtc/contracts/Dependencies/SafeERC20.sol";	// @audit-issue: Same library is also imported on: `['LeverageZapRouterBase']`, at lines: `[9]`

13:import {IStETH} from "./interface/IStETH.sol";	// @audit-issue: Same library is also imported on: `['LeverageZapRouterBase']`, at lines: `[12]`

15:import {IEbtcLeverageZapRouter} from "./interface/IEbtcLeverageZapRouter.sol";	// @audit-issue: Same library is also imported on: `['LeverageZapRouterBase']`, at lines: `[10]`
```
[7](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L7-L7), [8](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L8-L8), [10](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L10-L10), [11](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L11-L11), [13](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L13-L13), [15](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L15-L15), 


#### Recommendation

Review your Solidity contracts and eliminate duplicate import statements. Ensure each file is imported only once where it's needed. If you find the same import statement across multiple files, consider whether you can restructure your code to centralize common logic or dependencies, potentially through base contracts or libraries. Regularly conduct code audits or utilize linters and other static analysis tools to identify and resolve duplicate imports, thereby enhancing the clarity, structure, and security of your Solidity codebase.

### Consider using `delete` rather than assigning zero to clear values
The `delete` keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

106:        op.amountToTransferIn = 0;	// @audit-issue

175:        op.amountToTransferIn = 0;	// @audit-issue
```
[106](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L106-L106), [175](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L175-L175), 


#### Recommendation

When you need to clear or reset values in storage variables, consider using the `delete` keyword instead of manually assigning zero or other values. Using `delete` provides a more explicit and efficient way to clear storage variables. For example, instead of `myVariable = 0;`, you can use `delete myVariable;` to clear the value.

### Too long functions should be refactored
Functions with too many lines are difficult to understand. It is recommended to refactor complex functions into multiple shorter and easier to understand functions.


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

403:    function _adjustCdp(	// @audit-issue 64 lines
```
[403](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L403-L403), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

167:    function _doOperation(	// @audit-issue 77 lines
```
[167](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L167-L167), 


#### Recommendation

To address this issue, refactor long and complex functions into multiple shorter and more manageable functions. This will improve code readability and maintainability, making it easier to understand and maintain your smart contract.

### Expressions for `constant` values should use `immutable` rather than `constant`
While it does not save gas for some simple binary expressions because the compiler knows that developers often make this mistake, it's still best to use the right tool for the task at hand. There is a difference between `constant` variables and `immutable` variables, and they should each be used in their appropriate contexts. `constant`s should be used for literal values written into the code, and `immutable` variables should be used for expressions, or values calculated in, or passed into the `constructor`.

```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

37:    bytes32 constant FLASH_LOAN_SUCCESS = keccak256("ERC3156FlashBorrower.onFlashLoan");	// @audit-issue
```
[37](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L37-L37), 


#### Recommendation

To improve code readability and adhere to best practices, consider using `immutable` variables instead of `constant` for expressions or values calculated in, or passed into the constructor. While the compiler may handle this issue for simple binary expressions, it's essential to use the right tool for the task at hand. `constant` should be reserved for literal values written directly into the code, while `immutable` is better suited for dynamic values and expressions.

### Lines are too long
Usually lines in source code are limited to [80](https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width) characters. Today's screens are much larger so it's reasonable to stretch this in some cases. The solidity style guide recommends a maximumum line length of [120 characters](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length), so the lines below should be split when they reach that length.        self.impact_details = 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

30:    /// @param _ethMarginBalance The amount of margin deposit (converted from raw Ether) from the user, higher margin equals lower CR	// @audit-issue: Length of the line is: 134

73:    /// @param _wstEthMarginBalance The amount of margin deposit (converted from wrapped stETH) from the user, higher margin equals lower CR	// @audit-issue: Length of the line is: 141

124:    /// @param _wethMarginBalance The amount of margin deposit (converted from wrapped Ether) from the user, higher margin equals lower CR	// @audit-issue: Length of the line is: 139

167:    /// @param _stEthMarginAmount The amount of margin deposit (converted from staked Ether) from the user, higher margin equals lower CR	// @audit-issue: Length of the line is: 138

331:    /// @notice Function that allows various operations which might change both collateral (increase collateral with raw native Ether) and debt of a Cdp	// @audit-issue: Length of the line is: 153

349:    /// @notice Function that allows various operations which might change both collateral (increase collateral with wrapped Ether) and debt of a Cdp	// @audit-issue: Length of the line is: 150

367:    /// @notice Function that allows various operations which might change both collateral (increase collateral with wrapped Ether) and debt of a Cdp	// @audit-issue: Length of the line is: 150
```
[30](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L30-L30), [73](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L73-L73), [124](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L124-L124), [167](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L167-L167), [331](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L331-L331), [349](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L349-L349), [367](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L367-L367), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

24:/// @custom:known-issue Due to slippage some dust amounts for all intermediary tokens can be left, since there's no way to ask to sell all available	// @audit-issue: Length of the line is: 149

266:    /// @dev If you delegatecall into this, this will transfer the tokens to the caller of the DiamondLike (and not the contract)	// @audit-issue: Length of the line is: 130

463:        // But technically you could approve w/e you want here, this is fine because the contract is a router and will not hold user funds	// @audit-issue: Length of the line is: 139

476:        // val -> 0 -> 0 -> val means this is safe to repeat since even if full approve is unused, we always go back to 0 after	// @audit-issue: Length of the line is: 128

566:    /// @dev excessivelySafeCall to perform generic calls without getting gas bombed | useful if you don't care about return value	// @audit-issue: Length of the line is: 131
```
[24](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L24-L24), [266](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L266-L266), [463](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L463-L463), [476](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L476-L476), [566](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L566-L566), 


#### Recommendation

To adhere to coding standards and enhance code readability, consider splitting long lines of code when they approach the recommended maximum line length. In Solidity, a common guideline is to limit lines to a maximum of 120 characters. Splitting long lines can improve code maintainability and make it easier to understand.

### Dependence on external protocols
External protocols should be monitored as such dependencies may introduce vulnerabilities if a vulnerability is found /introduced in the external protocol

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

4:import {IPositionManagers} from "@ebtc/contracts/Interfaces/IBorrowerOperations.sol";	// @audit-issue

5:import {IPriceFeed} from "@ebtc/contracts/Interfaces/IPriceFeed.sol";	// @audit-issue

6:import {ICdpManagerData} from "@ebtc/contracts/Interfaces/ICdpManager.sol";	// @audit-issue

7:import {LeverageMacroBase} from "@ebtc/contracts/LeverageMacroBase.sol";	// @audit-issue

8:import {ReentrancyGuard} from "@ebtc/contracts/Dependencies/ReentrancyGuard.sol";	// @audit-issue

9:import {SafeERC20} from "@ebtc/contracts/Dependencies/SafeERC20.sol";	// @audit-issue

13:import {IERC20} from "@ebtc/contracts/Dependencies/IERC20.sol";	// @audit-issue
```
[4](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L4-L4), [5](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L5-L5), [6](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L6-L6), [7](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L7-L7), [8](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L8-L8), [9](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L9-L9), [13](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L13-L13), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

4:import {IERC3156FlashLender} from "@ebtc/contracts/Interfaces/IERC3156FlashLender.sol";	// @audit-issue

6:import {ICdpManagerData} from "@ebtc/contracts/Interfaces/ICdpManagerData.sol";	// @audit-issue

7:import {ICdpManager} from "@ebtc/contracts/Interfaces/ICdpManager.sol";	// @audit-issue

8:import {IBorrowerOperations} from "@ebtc/contracts/Interfaces/IBorrowerOperations.sol";	// @audit-issue

9:import {IPositionManagers} from "@ebtc/contracts/Interfaces/IPositionManagers.sol";	// @audit-issue

10:import {IERC20} from "@ebtc/contracts/Dependencies/IERC20.sol";	// @audit-issue

11:import {SafeERC20} from "@ebtc/contracts/Dependencies/SafeERC20.sol";	// @audit-issue

12:import {EbtcBase} from "@ebtc/contracts/Dependencies/EbtcBase.sol";	// @audit-issue
```
[4](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L4-L4), [6](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L6-L6), [7](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L7-L7), [8](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L8-L8), [9](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L9-L9), [10](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L10-L10), [11](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L11-L11), [12](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L12-L12), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

4:import {IPriceFeed} from "@ebtc/contracts/Interfaces/IPriceFeed.sol";	// @audit-issue

5:import {ICdpManagerData} from "@ebtc/contracts/Interfaces/ICdpManager.sol";	// @audit-issue

6:import {IBorrowerOperations, IPositionManagers} from "@ebtc/contracts/LeverageMacroBase.sol";	// @audit-issue

7:import {IERC20} from "@ebtc/contracts/Dependencies/IERC20.sol";	// @audit-issue
```
[4](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L4-L4), [5](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L5-L5), [6](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L6-L6), [7](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L7-L7), 


#### Recommendation

Regularly monitor and review the external protocols your Solidity contracts depend on for any updates or identified vulnerabilities. Consider implementing fallback mechanisms or contingency plans in your contracts to handle potential failures or compromises in these external protocols. Additionally, thoroughly audit and test the integration points with external protocols to ensure they adhere to your contract's security standards. Where possible, design your contract architecture to minimize reliance on external protocols or allow for upgradability in response to changes in these dependencies. Stay informed about developments in the protocols you depend on and actively participate in their community for early awareness of potential issues.

### Bug in Deduplication of Verbatim Blocks
The current Solidity version has the [Bug in Deduplication of Verbatim Blocks](https://soliditylang.org/blog/2023/11/08/verbatim-invalid-deduplication-bug/) issue.

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

It is recommended to follow the fix provided in the [link](https://soliditylang.org/blog/2023/11/08/verbatim-invalid-deduplication-bug/).

### Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`)
While the compiler knows to optimize away the exponentiation, it's still better coding practice to use idioms that do not require compiler optimization, if they exist


```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

19:    uint256 internal constant BPS = 10000;	// @audit-issue: This number `10000` can be written as `1e4`
```
[19](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L19-L19), 


#### Recommendation

Consider using scientific notation (e.g., `1e18`) instead of exponentiation (e.g., `10**18`) for better coding practice. While the compiler can optimize exponentiation, using scientific notation makes the code more readable and relies on standard idioms.

### Style Guide: Surround top level declarations in Solidity source with two blank lines.
1- Surround top level declarations in Solidity source with two blank lines.
2- Within a contract surround function declarations with a single blank line.


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

23:    ) LeverageZapRouterBase(params) { }
24:
25:    /// @dev Open a CDP with raw native Ether
26:    /// @param _debt The total expected debt for new CDP
27:    /// @param _upperHint The expected CdpId of neighboring higher ICR within SortedCdps, could be simply bytes32(0)
28:    /// @param _lowerHint The expected CdpId of neighboring lower ICR within SortedCdps, could be simply bytes32(0)
29:    /// @param _stEthLoanAmount The flash loan amount needed to open the leveraged Cdp position
30:    /// @param _ethMarginBalance The amount of margin deposit (converted from raw Ether) from the user, higher margin equals lower CR
31:    /// @param _stEthDepositAmount The total stETH collateral amount deposited (added) for the specified Cdp
32:    /// @param _positionManagerPermit PositionPermit required for Zap approved by calling user
33:    /// @param _tradeData DEX calldata for converting between debt and collateral	// @audit-issue: There should be single blank line between function declarations.
34:    function openCdpWithEth(

66:    }
67:
68:    /// @dev Open a CDP with wrapped staked Ether
69:    /// @param _debt The total expected debt for new CDP
70:    /// @param _upperHint The expected CdpId of neighboring higher ICR within SortedCdps, could be simply bytes32(0)
71:    /// @param _lowerHint The expected CdpId of neighboring lower ICR within SortedCdps, could be simply bytes32(0)
72:    /// @param _stEthLoanAmount The flash loan amount needed to open the leveraged Cdp position
73:    /// @param _wstEthMarginBalance The amount of margin deposit (converted from wrapped stETH) from the user, higher margin equals lower CR
74:    /// @param _stEthDepositAmount The total stETH collateral amount deposited (added) for the specified Cdp
75:    /// @param _positionManagerPermit PositionPermit required for Zap approved by calling user
76:    /// @param _tradeData DEX calldata for converting between debt and collateral	// @audit-issue: There should be single blank line between function declarations.
77:    function openCdpWithWstEth(

109:    }
110:
111:    /// @dev This is to allow wrapped ETH related Zap	// @audit-issue: There should be single blank line between function declarations.
112:    receive() external payable {

117:    }
118:
119:    /// @dev Open a CDP with wrapped Ether
120:    /// @param _debt The total expected debt for new CDP
121:    /// @param _upperHint The expected CdpId of neighboring higher ICR within SortedCdps, could be simply bytes32(0)
122:    /// @param _lowerHint The expected CdpId of neighboring lower ICR within SortedCdps, could be simply bytes32(0)
123:    /// @param _stEthLoanAmount The flash loan amount needed to open the leveraged Cdp position
124:    /// @param _wethMarginBalance The amount of margin deposit (converted from wrapped Ether) from the user, higher margin equals lower CR
125:    /// @param _stEthDepositAmount The total stETH collateral amount deposited (added) for the specified Cdp
126:    /// @param _positionManagerPermit PositionPermit required for Zap approved by calling user
127:    /// @param _tradeData DEX calldata for converting between debt and collateral	// @audit-issue: There should be single blank line between function declarations.
128:    function openCdpWithWrappedEth(

160:    }
161:
162:    /// @dev Open a CDP with staked Ether
163:    /// @param _debt The total expected debt for new CDP
164:    /// @param _upperHint The expected CdpId of neighboring higher ICR within SortedCdps, could be simply bytes32(0)
165:    /// @param _lowerHint The expected CdpId of neighboring lower ICR within SortedCdps, could be simply bytes32(0)
166:    /// @param _stEthLoanAmount The flash loan amount needed to open the leveraged Cdp position
167:    /// @param _stEthMarginAmount The amount of margin deposit (converted from staked Ether) from the user, higher margin equals lower CR
168:    /// @param _stEthDepositAmount The total stETH collateral amount deposited (added) for the specified Cdp
169:    /// @param _positionManagerPermit PositionPermit required for Zap approved by calling user
170:    /// @param _tradeData DEX calldata for converting between debt and collateral	// @audit-issue: There should be single blank line between function declarations.
171:    function openCdp(

248:    }
249:
250:    /// @dev Close a CDP with original collateral(stETH) returned to CDP owner
251:    /// @dev Note plain collateral(stETH) is returned no matter whatever asset is zapped in
252:    /// @param _cdpId The CdpId on which this operation is operated
253:    /// @param _positionManagerPermit PositionPermit required for Zap approved by calling user
254:    /// @param _tradeData DEX calldata for converting between debt and collateral	// @audit-issue: There should be single blank line between function declarations.
255:    function closeCdp(

261:    }
262:
263:    /// @dev Close a CDP with wrapped version of collateral(WstETH) returned to CDP owner
264:    /// @dev Note plain collateral(stETH) is returned no matter whatever asset is zapped in
265:    /// @param _cdpId The CdpId on which this operation is operated
266:    /// @param _positionManagerPermit PositionPermit required for Zap approved by calling user
267:    /// @param _tradeData DEX calldata for converting between debt and collateral	// @audit-issue: There should be single blank line between function declarations.
268:    function closeCdpForWstETH(

329:    }
330:
331:    /// @notice Function that allows various operations which might change both collateral (increase collateral with raw native Ether) and debt of a Cdp
332:    /// @param _cdpId The CdpId on which this operation is operated
333:    /// @param _params Parameters used for the adjust Cdp operation
334:    /// @param _positionManagerPermit PositionPermit required for Zap approved by calling user
335:    /// @param _tradeData DEX calldata for converting between debt and collateral	// @audit-issue: There should be single blank line between function declarations.
336:    function adjustCdpWithEth(

347:    }
348:
349:    /// @notice Function that allows various operations which might change both collateral (increase collateral with wrapped Ether) and debt of a Cdp
350:    /// @param _cdpId The CdpId on which this operation is operated
351:    /// @param _params Parameters used for the adjust Cdp operation
352:    /// @param _positionManagerPermit PositionPermit required for Zap approved by calling user
353:    /// @param _tradeData DEX calldata for converting between debt and collateral	// @audit-issue: There should be single blank line between function declarations.
354:    function adjustCdpWithWstEth(

365:    }
366:
367:    /// @notice Function that allows various operations which might change both collateral (increase collateral with wrapped Ether) and debt of a Cdp
368:    /// @param _cdpId The CdpId on which this operation is operated
369:    /// @param _params Parameters used for the adjust Cdp operation
370:    /// @param _positionManagerPermit PositionPermit required for Zap approved by calling user
371:    /// @param _tradeData DEX calldata for converting between debt and collateral	// @audit-issue: There should be single blank line between function declarations.
372:    function adjustCdpWithWrappedEth(

383:    }
384:
385:    /// @notice Function that allows various operations which might change both collateral and debt of a Cdp
386:    /// @param _cdpId The CdpId on which this operation is operated
387:    /// @param _params Parameters used for the adjust Cdp operation
388:    /// @param _positionManagerPermit PositionPermit required for Zap approved by calling user
389:    /// @param _tradeData DEX calldata for converting between debt and collateral	// @audit-issue: There should be single blank line between function declarations.
390:    function adjustCdp(
```
[33](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L23-L34), [76](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L66-L77), [111](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L109-L112), [127](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L117-L128), [170](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L160-L171), [254](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L248-L255), [267](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L261-L268), [335](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L329-L336), [353](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L347-L354), [371](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L365-L372), [389](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L383-L390), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

15:}
16:	// @audit-issue: There should be at least two blank lines between top level declarations.
17:abstract contract ZapRouterBase is IEbtcZapRouterBase {
```
[16](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L15-L17), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

46:    }
47:
48:    // Leverage Macro should receive a request and set that data
49:    // Then perform the request
50:	// @audit-issue: There should be single blank line between function declarations.
51:    constructor(

102:    }
103:
104:    /**
105:     * FL Setup
106:     *         - Validate Caller
107:     *
108:     *         FL
109:     *         - SwapsBefore
110:     *         - Operation
111:     *         - SwapsAfter
112:     *         - Repay
113:     *
114:     *         - Post Operation Checks
115:     *
116:     *         - Sweep
117:     */
118:    /// @notice Entry point for the Macro	// @audit-issue: There should be single blank line between function declarations.
119:    function doOperation(

156:    }
157:
158:    /// @notice Internal function used by derived contracts (i.e. EbtcZapRouter)
159:    /// @param flType flash loan type (eBTC, stETH or None)
160:    /// @param borrowAmount flash loan amount
161:    /// @param operation leverage macro operation
162:    /// @param postCheckType post operation check type
163:    /// @param checkParams post operation check params
164:    /// @param expectedCdpId pre-computed CDP ID used to run post operation checks
165:    /// @dev expectedCdpId is required for OpenCdp and OpenCdpFor, can be set to bytes32(0)
166:    /// for all other operations	// @audit-issue: There should be single blank line between function declarations.
167:    function _doOperation(

244:    }
245:
246:    /// @notice Sweep away tokens if they are stuck here	// @audit-issue: There should be single blank line between function declarations.
247:    function sweepToCaller() public {

263:    }
264:
265:    /// @notice Transfer an arbitrary token back to you
266:    /// @dev If you delegatecall into this, this will transfer the tokens to the caller of the DiamondLike (and not the contract)	// @audit-issue: There should be single blank line between function declarations.
267:    function sweepToken(address token, uint256 amount) public {

271:    }
272:
273:    /// @dev Assumes that
274:    ///     >= you prob use this one
275:    ///     <= if you don't need >= you go for lte
276:    ///     And if you really need eq, it's third	// @audit-issue: There should be single blank line between function declarations.
277:    function _doCheckValueType(CheckValueAndType memory check, uint256 valueToCheck) internal {

324:    }
325:
326:    /// @dev Must be memory since we had to decode it	// @audit-issue: There should be single blank line between function declarations.
327:    function _handleOperation(LeverageMacroOperation memory operation) internal {

385:    }
386:
387:    /// @notice Convenience function to parse bytes into LeverageMacroOperation data	// @audit-issue: There should be single blank line between function declarations.
388:    function decodeFLData(bytes calldata data) public view returns (LeverageMacroOperation memory) {

391:    }
392:
393:    /// @notice Proper Flashloan Callback handler	// @audit-issue: There should be single blank line between function declarations.
394:    function onFlashLoan(

429:    }
430:
431:    /// @dev Must be memory since we had to decode it	// @audit-issue: There should be single blank line between function declarations.
432:    function _doSwaps(SwapOperation[] memory swapData) internal {

441:    }
442:
443:    /// @dev Given a SwapOperation
444:    ///     Approves the `addressForApprove` for the exact amount
445:    ///     Calls `addressForSwap`
446:    ///     Resets the approval of `addressForApprove`
447:    ///     Performs validation via `_doSwapChecks`	// @audit-issue: There should be single blank line between function declarations.
448:    function _doSwap(SwapOperation memory swapData) internal {

481:    }
482:
483:    /// @dev Given `SwapCheck` performs validation on the state of this contract
484:    ///     A minOut Check	// @audit-issue: There should be single blank line between function declarations.
485:    function _doSwapChecks(SwapCheck[] memory swapChecks) internal {

497:    }
498:
499:    /// @dev Prevents doing arbitrary calls to protected targets	// @audit-issue: There should be single blank line between function declarations.
500:    function _ensureNotSystem(address addy) internal {

507:    }
508:
509:    /// @dev Must be memory since we had to decode it	// @audit-issue: There should be single blank line between function declarations.
510:    function _openCdpCallback(bytes memory data) internal virtual {

537:    }
538:
539:    /// @dev Must be memory since we had to decode it	// @audit-issue: There should be single blank line between function declarations.
540:    function _closeCdpCallback(bytes memory data) internal virtual {

545:    }
546:
547:    /// @dev Must be memory since we had to decode it	// @audit-issue: There should be single blank line between function declarations.
548:    function _adjustCdpCallback(bytes memory data) internal virtual {

564:    }
565:
566:    /// @dev excessivelySafeCall to perform generic calls without getting gas bombed | useful if you don't care about return value
567:    /// @notice Credits to: https://github.com/nomad-xyz/ExcessivelySafeCall/blob/main/src/ExcessivelySafeCall.sol	// @audit-issue: There should be single blank line between function declarations.
568:    function excessivelySafeCall(
```
[50](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L46-L51), [118](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L102-L119), [166](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L156-L167), [246](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L244-L247), [266](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L263-L267), [276](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L271-L277), [326](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L324-L327), [387](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L385-L388), [393](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L391-L394), [431](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L429-L432), [447](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L441-L448), [484](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L481-L485), [499](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L497-L500), [509](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L507-L510), [539](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L537-L540), [547](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L545-L548), [567](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L564-L568), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/latest/style-guide.html#blank-lines).

### Lack of index element validation in function
There's no validation to check whether the index element provided as an argument actually exists in the call. This omission could lead to unintended behavior if an element that does not exist in the call is passed to the function. The function should validate that the provided index element exists in the call before proceeding.

```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

436:            _doSwap(swapData[i]);	// @audit-issue

492:                        swapChecks[i].expectedMinOut,	// @audit-issue
```
[436](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L436-L436), [492](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L492-L492), 


#### Recommendation

Integrate explicit index validation checks at the beginning of functions that operate based on index elements. Use conditional statements to verify that the provided index falls within the valid range of existing elements. For array operations, ensure the index is less than the array's length. For mappings, consider additional logic to confirm the presence of a key. For example, in an array-based function:
```solidity
function getElementByIndex(uint256 index) public view returns (ElementType) {
    require(index < array.length, "Index out of bounds");
    return array[index];
}
```


### Contract should expose an `interface`
All `external`/`public` functions should extend an `interface`. This is useful to make sure that the whole API is extracted.


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

18:contract EbtcLeverageZapRouter is LeverageZapRouterBase {	// @audit-issue
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L18-L18), 


#### Recommendation

Consider defining an `interface` that includes all `external`/`public` functions of the contract. Exposing a well-defined interface helps ensure that the entire API is extracted and provides a clear and standardized way for other contracts or users to interact with your contract.

### Consider using named returns
Using named returns makes the code more self-documenting, makes it easier to fill out NatSpec, and in some cases can save gas. The cases below are where there currently is at most one return statement, which is ideal for named returns.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

61:    function owner() public override returns (address) {	// @audit-issue

257:    function _getPostCheckParams(
258:        bytes32 _cdpId,
259:        uint256 _debt,
260:        uint256 _totalCollateral,
261:        ICdpManagerData.Status _status,
262:        uint256 _collValidationBuffer
263:    ) internal view returns (PostCheckParams memory) {	// @audit-issue
```
[61](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L61-L61), [263](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L257-L263), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

14:    function MIN_CHANGE() external view returns (uint256);	// @audit-issue

34:    function _depositRawEthIntoLido(uint256 _initialETH) internal returns (uint256) {	// @audit-issue

43:    function _convertWrappedEthToStETH(uint256 _initialWETH) internal returns (uint256) {	// @audit-issue

54:    function _convertRawEthToStETH(uint256 _initialETH) internal returns (uint256) {	// @audit-issue

59:    function _convertWstEthToStETH(uint256 _initialWstETH) internal returns (uint256) {	// @audit-issue

107:    function _transferInitialStETHFromCaller(uint256 _initialStETH) internal returns (uint256) {	// @audit-issue

149:    function _getOwnerAddress(bytes32 cdpId) internal pure returns (address) {	// @audit-issue
```
[14](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L14-L14), [34](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L34-L34), [43](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L43-L43), [54](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L54-L54), [59](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L59-L59), [107](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L107-L107), [149](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L149-L149), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

15:    function Cdps(bytes32) external view returns (ICdpManagerData.Cdp memory);	// @audit-issue

39:    function owner() public virtual returns (address) {	// @audit-issue

388:    function decodeFLData(bytes calldata data) public view returns (LeverageMacroOperation memory) {	// @audit-issue

394:    function onFlashLoan(
395:        address initiator,
396:        address token,
397:        uint256 amount,
398:        uint256 fee,
399:        bytes calldata data
400:    ) external returns (bytes32) {	// @audit-issue

568:    function excessivelySafeCall(
569:        address _target,
570:        uint256 _gas,
571:        uint256 _value,
572:        uint16 _maxCopy,
573:        bytes memory _calldata
574:    ) internal returns (bool, bytes memory) {	// @audit-issue
```
[15](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L15-L15), [39](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L39-L39), [388](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L388-L388), [400](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L394-L400), [574](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L568-L574), 


#### Recommendation

Adopt named returns in your Solidity functions, especially in cases where functions contain a single return statement. This approach enhances code readability and documentation by making the return values clear and explicit. When defining your function, specify the return types with names, and manipulate these named variables directly within your function logic. Additionally, leverage named returns to streamline your NatSpec documentation, providing clear descriptions for each return variable. Evaluate your current contracts for opportunities to refactor functions to use named returns, prioritizing those with simple return patterns for immediate benefits in gas efficiency and code clarity.

### Hardcoded `address` should be avoided
It's better to declare the hardcoded addresses as `immutable` state variables, as this will facilitate deployment on other chains.


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

18:    address public constant NATIVE_ETH_ADDRESS = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;	// @audit-issue
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L18-L18), 


#### Recommendation

To enhance the flexibility and portability of your smart contract, consider declaring hardcoded addresses as `immutable` state variables. This allows for easier deployment on various chains and ensures that addresses can be updated if necessary without modifying the contract's code.

### Inefficient Array Usage
Use mappings instead of arrays for managing lists of data in order to conserve gas. Mappings are less expensive and more efficient for accessing any value without having to iterate through an array.

```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

432:    function _doSwaps(SwapOperation[] memory swapData) internal {	// @audit-issue

485:    function _doSwapChecks(SwapCheck[] memory swapChecks) internal {	// @audit-issue
```
[432](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L432-L432), [485](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L485-L485), 


#### Recommendation

In scenarios where data access efficiency is critical, prefer using mappings over arrays in Solidity contracts. Mappings offer more efficient and gas-effective data retrieval and updates, especially when dealing with large or frequently accessed datasets. Ensure to structure your data and choose keys thoughtfully to maximize the efficiency gains offered by mappings. While arrays might be suitable for ordered data or when the entire dataset needs to be iterated, for most other use cases, mappings are likely to be the more gas-efficient choice.

### Enum values should be used in place of constant array indexes
Consider using an `enum` instead of hardcoding an index access to make the code easier to understand.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

239:        swaps[0].tokenForSwap = _tokenIn;	// @audit-issue

240:        swaps[0].addressForApprove = DEX;	// @audit-issue

241:        swaps[0].exactApproveAmount = _tradeData.approvalAmount;	// @audit-issue

242:        swaps[0].addressForSwap = DEX;	// @audit-issue

243:        swaps[0].calldataForSwap = _tradeData.exchangeData;	// @audit-issue

245:            swaps[0].swapChecks = _getSwapChecks(_tokenOut, _tradeData.expectedMinOut);            	// @audit-issue

253:        checks[0].tokenToCheck = tokenToCheck;	// @audit-issue

254:        checks[0].expectedMinOut = expectedMinOut;	// @audit-issue
```
[239](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L239-L239), [240](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L240-L240), [241](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L241-L241), [242](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L242-L242), [243](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L243-L243), [245](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L245-L245), [253](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L253-L253), [254](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L254-L254), 


#### Recommendation

To improve code readability and maintainability, replace hardcoded array indexes with corresponding enum values. Enum values provide descriptive names for array elements, making your code more self-explanatory and reducing the risk of errors when working with arrays. This enhances the overall clarity and robustness of your smart contract code.

### Lack of specific import identifier
It is better to use `import {<identifier>} from "<file.sol>"` instead of `import "<file.sol>"` to improve readability and speed up the compilation time.

```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

4:import "./Interfaces/IBorrowerOperations.sol";	// @audit-issue

5:import "./Interfaces/IERC3156FlashLender.sol";	// @audit-issue

6:import "./Interfaces/IEBTCToken.sol";	// @audit-issue

7:import "./Interfaces/ICdpManager.sol";	// @audit-issue

8:import "./Interfaces/ISortedCdps.sol";	// @audit-issue

9:import "./Interfaces/IPriceFeed.sol";	// @audit-issue

10:import "./Dependencies/ICollateralToken.sol";	// @audit-issue

12:import "./Dependencies/SafeERC20.sol";	// @audit-issue
```
[4](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L4-L4), [5](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L5-L5), [6](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L6-L6), [7](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L7-L7), [8](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L8-L8), [9](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L9-L9), [10](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L10-L10), [12](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L12-L12), 


#### Recommendation

To improve code clarity and avoid naming conflicts, it's recommended to use specific import identifiers when importing from other contracts or libraries. Instead of using `import "<file.sol>";`, specify the desired identifiers using `import { <identifier1>, <identifier2> } from "<file.sol>";`. This not only enhances readability but also can speed up compilation times by only importing the necessary symbols.

### Consider using a `struct` rather than having many function input parameters
In Solidity, functions with a large number of input parameters can become cumbersome to manage and call. This can lead to readability issues and increased likelihood of errors, especially if the order of parameters is complex or not intuitive. To streamline this, consider consolidating multiple parameters into a single `struct`. This approach not only simplifies function signatures but also enhances code clarity. Structs allow for grouping related data together, making it easier to understand the relationship between parameters and manage them as a single entity.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

65:    function doOperation(
66:        FlashLoanType flType,
67:        uint256 borrowAmount,
68:        LeverageMacroOperation calldata operation,
69:        PostOperationCheck postCheckType,
70:        PostCheckParams calldata checkParams
71:    ) external override {

132:    function _adjustCdpOperation(
133:        bytes32 _cdpId,
134:        FlashLoanType _flType,
135:        uint256 _flAmount,
136:        AdjustCdpOperation memory _cdp,
137:        uint256 debt,
138:        uint256 coll,
139:        TradeData calldata _tradeData
140:    ) internal {

257:    function _getPostCheckParams(
258:        bytes32 _cdpId,
259:        uint256 _debt,
260:        uint256 _totalCollateral,
261:        ICdpManagerData.Status _status,
262:        uint256 _collValidationBuffer
263:    ) internal view returns (PostCheckParams memory) {
```
[74](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L65-L71), [163](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L132-L140), [274](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L257-L263), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

34:    function openCdpWithEth(
35:        uint256 _debt,
36:        bytes32 _upperHint,
37:        bytes32 _lowerHint,
38:        uint256 _stEthLoanAmount,
39:        uint256 _ethMarginBalance,
40:        uint256 _stEthDepositAmount,
41:        bytes calldata _positionManagerPermit,
42:        TradeData calldata _tradeData
43:    ) external payable returns (bytes32 cdpId) {

77:    function openCdpWithWstEth(
78:        uint256 _debt,
79:        bytes32 _upperHint,
80:        bytes32 _lowerHint,
81:        uint256 _stEthLoanAmount,
82:        uint256 _wstEthMarginBalance,
83:        uint256 _stEthDepositAmount,
84:        bytes calldata _positionManagerPermit,
85:        TradeData calldata _tradeData
86:    ) external returns (bytes32 cdpId) {

128:    function openCdpWithWrappedEth(
129:        uint256 _debt,
130:        bytes32 _upperHint,
131:        bytes32 _lowerHint,
132:        uint256 _stEthLoanAmount,
133:        uint256 _wethMarginBalance,
134:        uint256 _stEthDepositAmount,
135:        bytes calldata _positionManagerPermit,
136:        TradeData calldata _tradeData
137:    ) external returns (bytes32 cdpId) {

171:    function openCdp(
172:        uint256 _debt,
173:        bytes32 _upperHint,
174:        bytes32 _lowerHint,
175:        uint256 _stEthLoanAmount,
176:        uint256 _stEthMarginAmount,
177:        uint256 _stEthDepositAmount,
178:        bytes calldata _positionManagerPermit,
179:        TradeData calldata _tradeData
180:    ) external returns (bytes32 cdpId) {

205:    function _openCdp(
206:        uint256 _debt,
207:        bytes32 _upperHint,
208:        bytes32 _lowerHint,
209:        uint256 _stEthLoanAmount,
210:        uint256 _stEthMarginAmount,
211:        uint256 _stEthDepositAmount,
212:        bytes calldata _positionManagerPermit,
213:        TradeData calldata _tradeData
214:    ) internal nonReentrant returns (bytes32 cdpId) {

403:    function _adjustCdp(
404:        bytes32 _cdpId,
405:        AdjustCdpParams memory _params,
406:        bytes calldata _positionManagerPermit,
407:        TradeData calldata _tradeData,
408:        uint256 _zapStEthBalanceBefore
409:    ) internal nonReentrant {
```
[66](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L34-L43), [109](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L77-L86), [160](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L128-L137), [203](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L171-L180), [248](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L205-L214), [467](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L403-L409), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

51:    constructor(
52:        address _borrowerOperationsAddress,
53:        address _activePool,
54:        address _cdpManager,
55:        address _ebtc,
56:        address _coll,
57:        address _sortedCdps,
58:        bool _sweepToCaller
59:    ) {

119:    function doOperation(
120:        FlashLoanType flType,
121:        uint256 borrowAmount,
122:        LeverageMacroOperation calldata operation,
123:        PostOperationCheck postCheckType,
124:        PostCheckParams calldata checkParams
125:    ) external virtual {

167:    function _doOperation(
168:        FlashLoanType flType,
169:        uint256 borrowAmount,
170:        LeverageMacroOperation memory operation,
171:        PostOperationCheck postCheckType,
172:        PostCheckParams memory checkParams,
173:        bytes32 expectedCdpId
174:    ) internal {

394:    function onFlashLoan(
395:        address initiator,
396:        address token,
397:        uint256 amount,
398:        uint256 fee,
399:        bytes calldata data
400:    ) external returns (bytes32) {

568:    function excessivelySafeCall(
569:        address _target,
570:        uint256 _gas,
571:        uint256 _value,
572:        uint16 _maxCopy,
573:        bytes memory _calldata
574:    ) internal returns (bool, bytes memory) {
```
[68](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L51-L59), [156](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L119-L125), [244](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L167-L174), [429](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L394-L400), [604](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L568-L574), 


#### Recommendation

When faced with functions having numerous input parameters, refactor them to accept a `struct` instead. Define a `struct` that encapsulates all these parameters, thereby simplifying the function signature and improving code readability and maintainability. This method is particularly effective in complex functions or those with parameters that are logically related, making the code more intuitive and less error-prone.

### Some variables have a implicit default visibility
Consider always adding an explicit visibility modifier for variables, as the default is `internal`.

```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

37:    bytes32 constant FLASH_LOAN_SUCCESS = keccak256("ERC3156FlashBorrower.onFlashLoan");	// @audit-issue
```
[37](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L37-L37), 


#### Recommendation

Always add an explicit visibility modifier for variables to enhance code clarity and avoid potential issues. The default visibility for variables is `internal`, but specifying it explicitly makes your intentions clear.

### Unnecessary Use of override Keyword
In Solidity version 0.8.8 and later, the use of the override keyword becomes superfluous when a function is overriding solely from an interface and the function isn't present in multiple base contracts. Previously, the override keyword was required as an explicit indication to the compiler. However, this is no longer the case, and the extraneous use of the keyword can make the code less clean and more verbose.
Solidity documentation on [Function Overriding](https://docs.soliditylang.org/en/v0.8.20/contracts.html#function-overriding).


```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

61:    function owner() public override returns (address) {	// @audit-issue

65:    function doOperation(
66:        FlashLoanType flType,
67:        uint256 borrowAmount,
68:        LeverageMacroOperation calldata operation,
69:        PostOperationCheck postCheckType,
70:        PostCheckParams calldata checkParams
71:    ) external override {	// @audit-issue

276:    function _openCdpForCallback(bytes memory data) internal override {	// @audit-issue

294:    function _adjustCdpCallback(bytes memory data) internal override {	// @audit-issue
```
[61](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L61-L61), [71](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L65-L71), [276](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L276-L276), [294](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L294-L294), 


#### Recommendation

In Solidity versions 0.8.8 and later, the `override` keyword is no longer required for functions that are solely overriding from an interface and not present in multiple base contracts. Removing the unnecessary `override` keyword can make the code cleaner and less verbose.

### Use a single file for all system-wide constants
System-wide constants should be declared in a single file for better maintainability and readability. This contract seems to contain constants which could potentially be system-wide and could be better managed if they were centralized in a single location.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

18:    uint256 internal constant PRECISION = 1e18;	// @audit-issue

19:    uint256 internal constant BPS = 10000;	// @audit-issue
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L18-L18), [19](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L19-L19), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

18:    address public constant NATIVE_ETH_ADDRESS = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;	// @audit-issue

19:    uint256 public constant LIQUIDATOR_REWARD = 2e17;	// @audit-issue

20:    uint256 public constant MIN_NET_STETH_BALANCE = 2e18;	// @audit-issue
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L18-L18), [19](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L19-L19), [20](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L20-L20), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

37:    bytes32 constant FLASH_LOAN_SUCCESS = keccak256("ERC3156FlashBorrower.onFlashLoan");	// @audit-issue
```
[37](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L37-L37), 


#### Recommendation

Consider centralizing system-wide constants in a single file for better maintainability and readability. This practice makes it easier to manage and update constants across the contract and promotes consistency in your codebase.

### Interfaces should be defined in separate files from their usage
This issue arises when the interfaces are defined in the same files where they are used. They should be separated into different files for better readability and reusability.

In Solidity, interfaces are used to interact with contract abstractions. They define the functions and events that other contracts can call or listen to. While it is possible to define an interface within the same file where it is used, it is considered a good practice to separate the interface definition into its own file.

```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

13:interface IMinChangeGetter {	// @audit-issue used in lines: [28]
```
[13](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L13-L13), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

14:interface ICdpCdps {	// @audit-issue used in lines: [62]
```
[14](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L14-L14), 


#### Recommendation

To improve code organization and maintainability, it's recommended to define interfaces in separate files from their usage. This separation of concerns makes your codebase more modular and easier to understand. Create dedicated files for interface definitions, and then import and use these interfaces in the contracts that require them.

### Control structures do not follow the Solidity Style Guide
Refer to the [Solidity style guide - Control Structures](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#control-structures).

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

26:    constructor(
27:        IEbtcLeverageZapRouter.DeploymentParams memory params
28:    )	// @audit-issue
```
[28](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L26-L28), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

130:        if (
131:            operation.operationType == OperationType.OpenCdpOperation &&
132:            postCheckType != PostOperationCheck.none	// @audit-issue

139:        } else if (
140:            operation.operationType == OperationType.OpenCdpForOperation &&
141:            postCheckType != PostOperationCheck.none	// @audit-issue
```
[132](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L130-L132), [141](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L139-L141), 


#### Recommendation

Adhere to the Solidity style guide regarding control structures by avoiding the definition of multiple functions with identical names in a contract. Unique and descriptive function names improve code clarity and prevent potential confusion or errors. Consult [Solidity style guide - Control Structures](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#control-structures) for best practices.

### Consider making contracts `Upgradeable`
This allows for bugs to be fixed in production, at the expense of significantly increasing centralization.

```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

18:contract EbtcLeverageZapRouter is LeverageZapRouterBase {	// @audit-issue
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L18-L18), 


#### Recommendation

Assess the need for upgradeability in your Solidity contracts based on the project's requirements and lifecycle. If chosen, implement a well-known proxy pattern ensuring rigorous security and governance mechanisms are in place. Be aware of the increased centralization and plan accordingly to mitigate potential risks, such as through decentralized governance models or multi-sig control for upgrade decisions.

### Consider using descriptive `constants` when passing zero as a function argument
Passing zero as a function argument can sometimes result in a security issue (e.g. passing zero as the slippage parameter). Consider using a `constant` variable with a descriptive name, so it's clear that the argument is intentionally being used, and for the right reasons.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

224:            _getPostCheckParams(_cdpId, 0, 0, ICdpManagerData.Status.closedByOwner, 0),	// @audit-issue
```
[224](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L224-L224), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

464:        (bool success, ) = excessivelySafeCall(
465:            swapData.addressForSwap,
466:            gasleft(),
467:            0,	// @audit-issue
468:            0,
469:            swapData.calldataForSwap
470:        );

464:        (bool success, ) = excessivelySafeCall(
465:            swapData.addressForSwap,
466:            gasleft(),
467:            0,
468:            0,	// @audit-issue
469:            swapData.calldataForSwap
470:        );

477:        IERC20(swapData.tokenForSwap).safeApprove(swapData.addressForApprove, 0);	// @audit-issue
```
[467](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L464-L470), [468](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L464-L470), [477](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L477-L477), 


#### Recommendation

Replace direct usage of zero as a function argument with a well-named `constant`. For example, use something like `uint256 constant NO_SLIPPAGE = 0;` when zero is intended to signify 'no slippage'. This approach enhances code readability, reduces ambiguity, and helps ensure that the function is used correctly and for its intended purpose.

### Non-`external`/`public` function names should begin with an underscore
According to the Solidity Style Guide, Non-external/public function names should begin with an [underscore](https://docs.soliditylang.org/en/latest/style-guide.html#underscore-prefix-for-non-external-functions-and-variables)


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

568:    function excessivelySafeCall(	// @audit-issue
569:        address _target,
570:        uint256 _gas,
571:        uint256 _value,
572:        uint16 _maxCopy,
573:        bytes memory _calldata
574:    ) internal returns (bool, bytes memory) {
```
[568](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L568-L574), 


#### Recommendation

To adhere to the Solidity Style Guide, consider prefixing the names of non-`external`/`public` functions with an underscore (_). This naming convention enhances code readability and helps distinguish the visibility of functions.

### Large numeric literals should use underscores for readability
In Solidity, as in many programming languages, large numeric literals can be difficult to read and interpret at a glance, especially when they consist of many digits. To improve readability and reduce the likelihood of errors, it's beneficial to use underscores (`_`) as separators in these literals. This practice breaks down long numbers into smaller, more readable segments, similar to how commas are used in conventional numeric notation. For instance, `1000000` can be rewritten as `1_000_000`, making it immediately clear that it represents one million. This method of formatting large numbers enhances code clarity without affecting the actual value of the literals.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

19:    uint256 internal constant BPS = 10000;	// @audit-issue
```
[19](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L19-L19), 


#### Recommendation

For improved readability, consider using underscores in large numeric literals. This can make the numbers easier to parse and understand. For example, instead of writing `1000000000000000000`, you can write `1_000_000_000_000_000_000`.

### Variable names for `immutable`s should use CONSTANT_CASE
For `immutable` variable names, each word should use all capital letters, with underscores separating each word (CONSTANT_CASE)

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

21:    address public immutable theOwner;	// @audit-issue name should be: THE_OWNER

23:    uint256 public immutable zapFeeBPS;	// @audit-issue name should be: ZAP_FEE_B_P_S

24:    address public immutable zapFeeReceiver;	// @audit-issue name should be: ZAP_FEE_RECEIVER
```
[21](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L21-L21), [23](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L23-L23), [24](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L24-L24), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

23:    IERC20 public immutable wstEth;	// @audit-issue name should be: WST_ETH

24:    IStETH public immutable stEth;	// @audit-issue name should be: ST_ETH

25:    IERC20 public immutable wrappedEth;	// @audit-issue name should be: WRAPPED_ETH
```
[23](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L23-L23), [24](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L24-L24), [25](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L25-L25), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

29:    IBorrowerOperations public immutable borrowerOperations;	// @audit-issue name should be: BORROWER_OPERATIONS

30:    IActivePool public immutable activePool;	// @audit-issue name should be: ACTIVE_POOL

31:    ICdpCdps public immutable cdpManager;	// @audit-issue name should be: CDP_MANAGER

32:    IEBTCToken public immutable ebtcToken;	// @audit-issue name should be: EBTC_TOKEN

33:    ISortedCdps public immutable sortedCdps;	// @audit-issue name should be: SORTED_CDPS

34:    ICollateralToken public immutable stETH;	// @audit-issue name should be: ST_E_T_H

35:    bool internal immutable willSweep;	// @audit-issue name should be: WILL_SWEEP
```
[29](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L29-L29), [30](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L30-L30), [31](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L31-L31), [32](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L32-L32), [33](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L33-L33), [34](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L34-L34), [35](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L35-L35), 


#### Recommendation

When naming `immutable` variables, follow the CONSTANT_CASE convention, which means using all capital letters with underscores to separate words. This naming convention improves code readability and aligns with best practices for naming constants.

### Names of `private`/`internal` state variables should be prefixed with an underscore
It is recommended by the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#underscore-prefix-for-non-external-functions-and-variables).

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

18:    uint256 internal constant PRECISION = 1e18;	// @audit-issue name should be: _RECISION

19:    uint256 internal constant BPS = 10000;	// @audit-issue name should be: _PS
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L18-L18), [19](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L19-L19), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

35:    bool internal immutable willSweep;	// @audit-issue name should be: _illSweep
```
[35](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L35-L35), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### Using zero as a parameter
Taking 0 as a valid argument in Solidity without checks can lead to severe security issues. A historical example is the infamous 0x0 address bug where numerous tokens were lost. This happens because '0' can be interpreted as an uninitialized address, leading to transfers to the '0x0' address, effectively burning tokens. Moreover, 0 as a denominator in division operations would cause a runtime exception. It's also often indicative of a logical error in the caller's code. It's important to always validate input and handle edge cases like 0 appropriately. Use `require()` statements to enforce conditions and provide clear error messages to facilitate debugging and safer code.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

224:            _getPostCheckParams(_cdpId, 0, 0, ICdpManagerData.Status.closedByOwner, 0),	// @audit-issue
```
[224](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L224-L224), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

464:        (bool success, ) = excessivelySafeCall(
465:            swapData.addressForSwap,
466:            gasleft(),
467:            0,	// @audit-issue
468:            0,
469:            swapData.calldataForSwap
470:        );

464:        (bool success, ) = excessivelySafeCall(
465:            swapData.addressForSwap,
466:            gasleft(),
467:            0,
468:            0,	// @audit-issue
469:            swapData.calldataForSwap
470:        );
```
[467](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L464-L470), [468](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L464-L470), 


#### Recommendation

Implement stringent checks in your Solidity contracts to validate inputs and avoid the unsafe use of zero. Utilize `require()` statements to ensure that critical parameters, such as addresses and denominators in division operations, are not zero. Provide clear and informative error messages in these `require()` statements to aid in debugging and to clearly communicate the nature of the error. This practice not only enhances the security of your contract but also helps prevent logical errors and facilitates safer interactions with the contract.

### Revert statements within external and public functions can be used to perform DOS attacks
In Solidity, `revert` statements are used to undo changes and throw an exception when certain conditions are not met. However, in public and external functions, improper use of `revert` can be exploited for Denial of Service (DoS) attacks. An attacker can intentionally trigger these `revert' conditions, causing legitimate transactions to consistently fail. For example, if a function relies on specific conditions from user input or contract state, an attacker could manipulate these to continually force `revert`s, blocking the function's execution. Therefore, it's crucial to design contract logic to handle exceptions properly and avoid scenarios where `revert` can be predictably triggered by malicious actors. This includes careful input validation and considering alternative design patterns that are less susceptible to such abuses.

```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

406:            require(	// @audit-issue

412:            require(	// @audit-issue
```
[406](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L406-L406), [412](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L412-L412), 


#### Recommendation

Design your Solidity contract's public and external functions with care to mitigate the risk of DoS attacks via `revert` statements. Implement robust input validation to ensure inputs are within expected bounds and conditions. Consider alternative logic or design patterns that reduce the reliance on `revert` for critical operations, particularly those that can be influenced externally. Evaluate the use of modifiers, try-catch blocks, or state checks that allow for safer handling of conditions and exceptions. Ensure that your contract's critical functionality remains accessible and resilient against potential abuse of `revert` behavior by malicious actors.

### Consider adding emergency-stop functionality
In the event of a security breach or any unforeseen emergency, swiftly suspending all protocol operations becomes crucial. Having a mechanism in place to halt all functions collectively, instead of pausing individual contracts separately, substantially enhances the efficiency of mitigating ongoing attacks or vulnerabilities. This not only quickens the response time to potential threats but also reduces operational stress during these critical periods. Therefore, consider integrating a 'circuit breaker' or 'emergency stop' function into the smart contract system architecture. Such a feature would provide the capability to suspend the entire protocol instantly, which could prove invaluable during a time-sensitive crisis management situation.

```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

18:contract EbtcLeverageZapRouter is LeverageZapRouterBase {	// @audit-issue
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L18-L18), 


#### Recommendation

Implement an emergency-stop feature in your Solidity contract system to enhance security and crisis response capabilities. This can be achieved through a 'circuit breaker' pattern, where a central switch or set of conditions can instantly suspend critical operations across the contract ecosystem. Ensure that this mechanism is accessible to authorized parties only, such as contract administrators or a decentralized governance system. Design the emergency-stop functionality to be transparent and auditable, with clear conditions and processes for activation and deactivation. Regularly test and audit this feature to ensure its reliability and effectiveness in potential emergency situations.

### Assembly block creates dirty bits
Manipulating data directly at the free memory pointer location without subsequently adjusting the pointer can lead to unwanted data remnants, or "dirty bits", in that memory spot. This can cause challenges for the Solidity optimizer, making it difficult to determine if memory cleaning is required before reuse, potentially resulting in less efficient optimization. To mitigate this issue, it's advised to always update the free memory pointer following any data write operation. Furthermore, using the `assembly ("memory-safe") { ... }` annotation will clearly indicate to the optimizer the sections of your code that are memory-safe, improving code efficiency and reducing the potential for errors.

```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

583:        assembly {	// @audit-issue
584:            _success := call(
585:                _gas, // gas
586:                _target, // recipient
587:                _value, // ether value
588:                add(_calldata, 0x20), // inloc
589:                mload(_calldata), // inlen
590:                0, // outloc
591:                0 // outlen
592:            )
593:            // limit our copy to 256 bytes
594:            _toCopy := returndatasize()
595:            if gt(_toCopy, _maxCopy) {
596:                _toCopy := _maxCopy
597:            }
598:            // Store the length of the copied bytes
599:            mstore(_returnData, _toCopy)
600:            // copy the bytes from returndata[0:_toCopy]
601:            returndatacopy(add(_returnData, 0x20), 0, _toCopy)
602:        }
```
[583](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L583-L602), 


#### Recommendation

Always update the free memory pointer after writing data in Solidity assembly blocks to ensure memory cleanliness. This practice prevents the creation of 'dirty bits' and aids the Solidity optimizer in performing efficient memory management. Additionally, consider using the `assembly ("memory-safe") { ... }` annotation for sections of your code that are memory-safe. This annotation clearly communicates to the optimizer which parts of your assembly code are handling memory correctly, leading to better optimization and reduced risk of memory-related errors. Regularly review and audit your assembly blocks for proper memory handling to maintain the efficiency and reliability of your Solidity contracts.

### For extended 'using-for' usage, use the latest pragma version
Solidity versions of 0.8.13 or above can make use of enhanced using-for notation within contracts.

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
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

26:abstract contract LeverageMacroBase {	// @audit-issue
```
[26](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L26-L26), 


#### Recommendation

Update your Solidity contracts to use the latest pragma version, ideally 0.8.13 or higher, to leverage the enhanced 'using-for' notation. This upgrade will enable you to apply library functions to user-defined types more effectively, enhancing the functionality and readability of your contracts. Make sure to thoroughly test your contracts after upgrading to ensure compatibility and to take full advantage of the latest Solidity features and improvements. Regularly keep your contracts updated with the latest Solidity versions to stay aligned with the evolving capabilities and best practices of the language.

### Cyclomatic complexity in functions
Cyclomatic complexity is a software metric used to measure the complexity of a program. It quantifies the number of linearly independent paths through a program's source code, giving an idea of how complex the control flow is. High cyclomatic complexity may indicate a higher risk of defects and can make the code harder to understand, test, and maintain. It often suggests that a function or method is trying to do too much, and a refactor might be needed. By breaking down complex functions into smaller, more focused pieces, you can improve readability, ease of testing, and overall maintainability.

```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

167:    function _doOperation(	// @audit-issue
168:        FlashLoanType flType,
169:        uint256 borrowAmount,
170:        LeverageMacroOperation memory operation,
171:        PostOperationCheck postCheckType,
172:        PostCheckParams memory checkParams,
173:        bytes32 expectedCdpId
174:    ) internal {
175:        // Call FL Here, then the stuff below needs to happen inside the FL
176:        if (operation.amountToTransferIn > 0) {
177:            IERC20(operation.tokenToTransferIn).safeTransferFrom(
178:                msg.sender,
179:                address(this),
180:                operation.amountToTransferIn
181:            );
182:        }
183:
184:        // Take eBTC or stETH FlashLoan
185:        if (flType == FlashLoanType.eBTC) {
186:            IERC3156FlashLender(address(borrowerOperations)).flashLoan(
187:                IERC3156FlashBorrower(address(this)),
188:                address(ebtcToken),
189:                borrowAmount,
190:                abi.encode(operation)
191:            );
192:        } else if (flType == FlashLoanType.stETH) {
193:            IERC3156FlashLender(address(activePool)).flashLoan(
194:                IERC3156FlashBorrower(address(this)),
195:                address(stETH),
196:                borrowAmount,
197:                abi.encode(operation)
198:            );
199:        } else {
200:            // No leverage, just do the operation
201:            _handleOperation(operation);
202:        }
203:
204:        /**
205:         * POST CALL CHECK FOR CREATION
206:         */
207:        if (postCheckType == PostOperationCheck.openCdp) {
208:            // Check for param details
209:            ICdpManagerData.Cdp memory cdpInfo = cdpManager.Cdps(expectedCdpId);
210:            _doCheckValueType(checkParams.expectedDebt, cdpInfo.debt);
211:            _doCheckValueType(checkParams.expectedCollateral, cdpInfo.coll);
212:            require(
213:                cdpInfo.status == checkParams.expectedStatus,
214:                "!LeverageMacroReference: openCDP status check"
215:            );
216:        }
217:
218:        // Update CDP, Ensure the stats are as intended
219:        if (postCheckType == PostOperationCheck.cdpStats) {
220:            ICdpManagerData.Cdp memory cdpInfo = cdpManager.Cdps(checkParams.cdpId);
221:
222:            _doCheckValueType(checkParams.expectedDebt, cdpInfo.debt);
223:            _doCheckValueType(checkParams.expectedCollateral, cdpInfo.coll);
224:            require(
225:                cdpInfo.status == checkParams.expectedStatus,
226:                "!LeverageMacroReference: adjustCDP status check"
227:            );
228:        }
229:
230:        // Post check type: Close, ensure it has the status we want
231:        if (postCheckType == PostOperationCheck.isClosed) {
232:            ICdpManagerData.Cdp memory cdpInfo = cdpManager.Cdps(checkParams.cdpId);
233:
234:            require(
235:                cdpInfo.status == checkParams.expectedStatus,
236:                "!LeverageMacroReference: closeCDP status check"
237:            );
238:        }
239:
240:        // Sweep here if it's Reference, do not if it's delegate
241:        if (willSweep) {
242:            sweepToCaller();
243:        }
244:    }

327:    function _handleOperation(LeverageMacroOperation memory operation) internal {	// @audit-issue
328:        uint256 beforeSwapsLength = operation.swapsBefore.length;
329:        if (beforeSwapsLength > 0) {
330:            _doSwaps(operation.swapsBefore);
331:        }
332:
333:        // Based on the type we do stuff
334:        if (operation.operationType == OperationType.OpenCdpOperation) {
335:            _openCdpCallback(operation.OperationData);
336:        } else if (operation.operationType == OperationType.OpenCdpForOperation) {
337:            _openCdpForCallback(operation.OperationData);
338:        } else if (operation.operationType == OperationType.CloseCdpOperation) {
339:            _closeCdpCallback(operation.OperationData);
340:        } else if (operation.operationType == OperationType.AdjustCdpOperation) {
341:            _adjustCdpCallback(operation.OperationData);
342:        } else if (operation.operationType == OperationType.ClaimSurplusOperation) {
343:            _claimSurplusCallback();
344:        }
345:
346:        uint256 afterSwapsLength = operation.swapsAfter.length;
347:        if (afterSwapsLength > 0) {
348:            _doSwaps(operation.swapsAfter);
349:        }
350:    }
```
[167](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L167-L244), [327](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L327-L350), 


#### Recommendation

Regularly analyze your Solidity contracts for functions with high cyclomatic complexity. Consider refactoring these functions into smaller, more focused units. This could involve breaking down complex functions into multiple simpler functions, reducing the number of conditional branches, or simplifying logic where possible. Such refactoring improves the readability and testability of your code and reduces the likelihood of defects. Additionally, simpler functions are easier to audit and maintain, enhancing the overall security and robustness of your contract. Employ tools or metrics to periodically assess the complexity of your functions as part of your development and maintenance processes.

### Consider only defining one library/interface/contract per sol file
Combining multiple libraries, interfaces, or contracts in a single .sol file can lead to clutter, reduced readability, and versioning issues. Resolution: Adopt the best practice of defining only one library, interface, or contract per Solidity file. This modular approach enhances clarity, simplifies unit testing, and streamlines code review. Furthermore, segregating components makes version management easier, as updates to one component won't necessitate changes to a file housing multiple unrelated components. Structured file management can further assist in avoiding naming collisions and ensure smoother integration into larger systems or DApps.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

4:import {IPositionManagers} from "@ebtc/contracts/Interfaces/IBorrowerOperations.sol";	// @audit-issue
5:import {IPriceFeed} from "@ebtc/contracts/Interfaces/IPriceFeed.sol";
6:import {ICdpManagerData} from "@ebtc/contracts/Interfaces/ICdpManager.sol";
7:import {LeverageMacroBase} from "@ebtc/contracts/LeverageMacroBase.sol";
8:import {ReentrancyGuard} from "@ebtc/contracts/Dependencies/ReentrancyGuard.sol";
9:import {SafeERC20} from "@ebtc/contracts/Dependencies/SafeERC20.sol";
10:import {IEbtcLeverageZapRouter} from "./interface/IEbtcLeverageZapRouter.sol";
11:import {ZapRouterBase} from "./ZapRouterBase.sol";
12:import {IStETH} from "./interface/IStETH.sol";
13:import {IERC20} from "@ebtc/contracts/Dependencies/IERC20.sol";
```
[4](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L4-L13), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

4:import {IERC3156FlashLender} from "@ebtc/contracts/Interfaces/IERC3156FlashLender.sol";	// @audit-issue
5:import {LeverageZapRouterBase} from "./LeverageZapRouterBase.sol";
6:import {ICdpManagerData} from "@ebtc/contracts/Interfaces/ICdpManagerData.sol";
7:import {ICdpManager} from "@ebtc/contracts/Interfaces/ICdpManager.sol";
8:import {IBorrowerOperations} from "@ebtc/contracts/Interfaces/IBorrowerOperations.sol";
9:import {IPositionManagers} from "@ebtc/contracts/Interfaces/IPositionManagers.sol";
10:import {IERC20} from "@ebtc/contracts/Dependencies/IERC20.sol";
11:import {SafeERC20} from "@ebtc/contracts/Dependencies/SafeERC20.sol";
12:import {EbtcBase} from "@ebtc/contracts/Dependencies/EbtcBase.sol";
13:import {IStETH} from "./interface/IStETH.sol";
14:import {IWrappedETH} from "./interface/IWrappedETH.sol";
15:import {IEbtcLeverageZapRouter} from "./interface/IEbtcLeverageZapRouter.sol";
16:import {IWstETH} from "./interface/IWstETH.sol";
```
[4](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L4-L16), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

4:import {IPriceFeed} from "@ebtc/contracts/Interfaces/IPriceFeed.sol";	// @audit-issue
5:import {ICdpManagerData} from "@ebtc/contracts/Interfaces/ICdpManager.sol";
6:import {IBorrowerOperations, IPositionManagers} from "@ebtc/contracts/LeverageMacroBase.sol";
7:import {IERC20} from "@ebtc/contracts/Dependencies/IERC20.sol";
8:import {IEbtcZapRouterBase} from "./interface/IEbtcZapRouterBase.sol";
9:import {IWrappedETH} from "./interface/IWrappedETH.sol";
10:import {IStETH} from "./interface/IStETH.sol";
11:import {IWstETH} from "./interface/IWstETH.sol";
```
[4](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L4-L11), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

4:import "./Interfaces/IBorrowerOperations.sol";	// @audit-issue
5:import "./Interfaces/IERC3156FlashLender.sol";
6:import "./Interfaces/IEBTCToken.sol";
7:import "./Interfaces/ICdpManager.sol";
8:import "./Interfaces/ISortedCdps.sol";
9:import "./Interfaces/IPriceFeed.sol";
10:import "./Dependencies/ICollateralToken.sol";
11:import {ICdpManagerData} from "./Interfaces/ICdpManagerData.sol";
12:import "./Dependencies/SafeERC20.sol";
```
[4](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L4-L12), 


#### Recommendation

Adopt a modular file structure in your Solidity projects by defining only one library, interface, or contract per file. This approach significantly enhances the clarity and readability of your code, making it easier to manage, test, and review. It also simplifies version control, as updates to individual components are isolated to their respective files, reducing the risk of unintended side effects. Organize your project directory to reflect this structure, with a clear naming convention that matches file names to their contained contracts, libraries, or interfaces. This organization not only prevents naming collisions but also facilitates smoother integration into larger systems or decentralized applications (DApps).

### Reduce deployment costs by tweaking contracts' metadata
When solidity generates the bytecode for the smart contract to be deployed, it appends metadata about the compilation at the end of the bytecode.
By default, the solidity compiler appends metadata at the end of the “actual” initcode, which gets stored to the blockchain when the constructor finishes executing.
Consider tweaking the metadata to avoid this unnecessary allocation. A full guide can be found [here](https://www.rareskills.io/post/solidity-metadata).
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L1), 


#### Recommendation

See [this](https://www.rareskills.io/post/solidity-metadata) link, at its bottom, for full details

### All verbatim blocks are considered identical by deduplicator and can incorrectly be unified
The Solidity Team reported a bug on October 24, 2023, affecting Yul code using the verbatim builtin, specifically in the Block Deduplicator optimizer step. This bug, present since Solidity version 0.8.5, caused incorrect deduplication of verbatim assembly items surrounded by identical opcodes, considering them identical regardless of their data. The bug was confined to pure Yul compilation with optimization enabled and was unlikely to be exploited as an attack vector. The conditions triggering the bug were very specific, and its occurrence was deemed to have a low likelihood. The bug was rated with an overall low score due to these factors.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L1), 


#### Recommendation

Review and assess any Solidity contracts, especially those involving Yul code, that may be impacted by this deduplication bug. If your contracts rely on the Block Deduplicator optimizer and use verbatim blocks in a way that could be affected by this issue, consider updating your Solidity version to one where this bug is fixed, or adjust your contract to avoid this specific scenario. Stay informed about updates from the Solidity Team regarding this and similar issues, and regularly update your Solidity compiler to the latest version to benefit from bug fixes and optimizations. Given the specific and limited nature of this bug, its impact may be minimal, but caution is advised for contracts with complex assembly code or those heavily reliant on optimizer behaviors.

### Consider adding formal verification proofs
Formal verification is the act of proving or disproving the correctness of intended algorithms underlying a system with respect to a certain formal specification/property/invariant, using formal methods of mathematics.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L1), 


#### Recommendation

Consider integrating formal verification into your Solidity contract development process. This can be done by defining formal specifications and properties that your contract should adhere to and using mathematical methods to verify these aspects. Tools and platforms like Certora Prover, Scribble, or OpenZeppelin's test environment can assist in this process. Formal verification should complement traditional testing and auditing methods, offering an additional layer of security assurance. Keep in mind that formal verification requires a thorough understanding of mathematical logic and contract specifications, so it may necessitate additional resources or expertise. Nevertheless, the investment in formal verification can significantly enhance the trustworthiness and robustness of your smart contracts.

### Contracts should have full test coverage
While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L1), 


#### Recommendation

Consider writing test cases.

### Large or complicated code bases should implement invariant tests
This includes: large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts. Invariant fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and invariant fuzzers may help significantly.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L1), 


#### Recommendation

Consider writing invariant test cases.

### Consider adding formal verification proofs
Consider using formal verification to mathematically prove that your code does what is intended, and does not have any edge cases with unexpected behavior. The solidity compiler itself has this functionality [built in](https://docs.soliditylang.org/en/latest/smtchecker.html#smtchecker-and-formal-verification)
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L1), 


#### Recommendation

Consider using formal verification.

### TODO Left in the code
`TODO` comments are mark areas of code that need attention or completion. These comments serve as reminders for unfinished tasks and can be helpful during the development phase. However, if left untouched in production code, these `TODO` statements can introduce security vulnerabilities and impact the overall security of a smart contract.

```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

37:        // TODO call submit() with a referral?	// @audit-issue
```
[37](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L37-L37), 


#### Recommendation

It's important to remove `TODO` comments from production code to avoid potential security vulnerabilities. These comments should be addressed and resolved during the development phase.

### NatSpec: Contract declarations should have `@author` tag
In the world of decentralized code, giving credit is key. NatSpec's `@author` tag acknowledges the minds behind the code. It appears this Solidity contract omits the `@author` directive in its NatSpec annotations. Properly attributing code to its contributors not only recognizes effort but also aids in establishing trust and credibility. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

15:abstract contract LeverageZapRouterBase is ZapRouterBase, LeverageMacroBase, ReentrancyGuard, IEbtcLeverageZapRouter {	// @audit-issue missing `@author` tag
```
[15](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L15-L15), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

18:contract EbtcLeverageZapRouter is LeverageZapRouterBase {	// @audit-issue missing `@author` tag
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L18-L18), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

13:interface IMinChangeGetter {	// @audit-issue missing `@author` tag

17:abstract contract ZapRouterBase is IEbtcZapRouterBase {	// @audit-issue missing `@author` tag
```
[13](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L13-L13), [17](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L17-L17), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

14:interface ICdpCdps {	// @audit-issue missing `@author` tag

26:abstract contract LeverageMacroBase {	// @audit-issue missing `@author` tag
```
[14](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L14-L14), [26](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L26-L26), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Contract declarations should have `@dev` tag
NatSpec comments are a critical part of Solidity's documentation system, designed to help developers and others understand the behavior and purpose of a contract. The `@dev` tag, in particular, provides context and insight into the contract's development considerations. A missing `@dev` comment can lead to misunderstandings about the contract, making it harder for others to contribute to or use the contract effectively. Therefore, it's highly recommended to include `@dev` comments in the documentation to enhance code readability and maintainability. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

15:abstract contract LeverageZapRouterBase is ZapRouterBase, LeverageMacroBase, ReentrancyGuard, IEbtcLeverageZapRouter {	// @audit-issue missing `@dev` tag
```
[15](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L15-L15), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

18:contract EbtcLeverageZapRouter is LeverageZapRouterBase {	// @audit-issue missing `@dev` tag
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L18-L18), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

13:interface IMinChangeGetter {	// @audit-issue missing `@dev` tag

17:abstract contract ZapRouterBase is IEbtcZapRouterBase {	// @audit-issue missing `@dev` tag
```
[13](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L13-L13), [17](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L17-L17), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

14:interface ICdpCdps {	// @audit-issue missing `@dev` tag
```
[14](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L14-L14), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Contract declarations should have `@notice` tag
The `@notice` is used to explain to users what the contract does. The compiler interprets `///` or `/**` comments [as this tag](https://docs.soliditylang.org/en/latest/natspec-format.html#tags) if one wasn't explicitly provided.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

15:abstract contract LeverageZapRouterBase is ZapRouterBase, LeverageMacroBase, ReentrancyGuard, IEbtcLeverageZapRouter {	// @audit-issue missing `@notice` tag
```
[15](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L15-L15), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

18:contract EbtcLeverageZapRouter is LeverageZapRouterBase {	// @audit-issue missing `@notice` tag
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L18-L18), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

13:interface IMinChangeGetter {	// @audit-issue missing `@notice` tag

17:abstract contract ZapRouterBase is IEbtcZapRouterBase {	// @audit-issue missing `@notice` tag
```
[13](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L13-L13), [17](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L17-L17), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

14:interface ICdpCdps {	// @audit-issue missing `@notice` tag
```
[14](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L14-L14), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Contract declarations should have `@title` tag
The `@title` is used to explain to users what the contract does. The compiler interprets `///` or `/**` comments [as this tag](https://docs.soliditylang.org/en/latest/natspec-format.html#tags) if one wasn't explicitly provided.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

15:abstract contract LeverageZapRouterBase is ZapRouterBase, LeverageMacroBase, ReentrancyGuard, IEbtcLeverageZapRouter {	// @audit-issue missing `@title` tag
```
[15](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L15-L15), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

18:contract EbtcLeverageZapRouter is LeverageZapRouterBase {	// @audit-issue missing `@title` tag
```
[18](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L18-L18), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

13:interface IMinChangeGetter {	// @audit-issue missing `@title` tag

17:abstract contract ZapRouterBase is IEbtcZapRouterBase {	// @audit-issue missing `@title` tag
```
[13](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L13-L13), [17](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L17-L17), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

14:interface ICdpCdps {	// @audit-issue missing `@title` tag
```
[14](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L14-L14), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Use `@inheritdoc` for overriden functions.
Natural Specification (NatSpec) comments are crucial for understanding the role of function arguments in your Solidity code. Including @param tags will not only improve your code's readability but also its maintainability by clearly defining each argument's purpose. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

61:    function owner() public override returns (address) {	// @audit-issue missing `@inheritdoc` tag

65:    function doOperation(	// @audit-issue missing `@inheritdoc` tag

276:    function _openCdpForCallback(bytes memory data) internal override {	// @audit-issue missing `@inheritdoc` tag

294:    function _adjustCdpCallback(bytes memory data) internal override {	// @audit-issue missing `@inheritdoc` tag
```
[61](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L61-L61), [65](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L65-L65), [276](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L276-L276), [294](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L294-L294), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Function `@return` tag is missing
Natural Specification (NatSpec) comments are crucial for understanding the role of function arguments in your Solidity code. Including `@return` tag will not only improve your code's readability but also its maintainability by clearly defining each argument's purpose. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

61:    function owner() public override returns (address) {	// @audit-issue missing `@return` tag

100:    function _getAdjustCdpParams(	// @audit-issue missing `@return` tag

232:    function _getSwapOperations(	// @audit-issue missing `@return` tag

249:    function _getSwapChecks(address tokenToCheck, uint256 expectedMinOut) 	// @audit-issue missing `@return` tag

257:    function _getPostCheckParams(	// @audit-issue missing `@return` tag
```
[61](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L61-L61), [100](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L100-L100), [232](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L232-L232), [249](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L249-L249), [257](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L257-L257), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

34:    function openCdpWithEth(	// @audit-issue missing `@return` tag

77:    function openCdpWithWstEth(	// @audit-issue missing `@return` tag

128:    function openCdpWithWrappedEth(	// @audit-issue missing `@return` tag

171:    function openCdp(	// @audit-issue missing `@return` tag

205:    function _openCdp(	// @audit-issue missing `@return` tag
```
[34](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L34-L34), [77](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L77-L77), [128](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L128-L128), [171](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L171-L171), [205](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L205-L205), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

14:    function MIN_CHANGE() external view returns (uint256);	// @audit-issue missing `@return` tag

34:    function _depositRawEthIntoLido(uint256 _initialETH) internal returns (uint256) {	// @audit-issue missing `@return` tag

43:    function _convertWrappedEthToStETH(uint256 _initialWETH) internal returns (uint256) {	// @audit-issue missing `@return` tag

54:    function _convertRawEthToStETH(uint256 _initialETH) internal returns (uint256) {	// @audit-issue missing `@return` tag

59:    function _convertWstEthToStETH(uint256 _initialWstETH) internal returns (uint256) {	// @audit-issue missing `@return` tag

107:    function _transferInitialStETHFromCaller(uint256 _initialStETH) internal returns (uint256) {	// @audit-issue missing `@return` tag

149:    function _getOwnerAddress(bytes32 cdpId) internal pure returns (address) {	// @audit-issue missing `@return` tag
```
[14](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L14-L14), [34](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L34-L34), [43](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L43-L43), [54](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L54-L54), [59](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L59-L59), [107](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L107-L107), [149](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L149-L149), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

15:    function Cdps(bytes32) external view returns (ICdpManagerData.Cdp memory);	// @audit-issue missing `@return` tag

39:    function owner() public virtual returns (address) {	// @audit-issue missing `@return` tag

388:    function decodeFLData(bytes calldata data) public view returns (LeverageMacroOperation memory) {	// @audit-issue missing `@return` tag

394:    function onFlashLoan(	// @audit-issue missing `@return` tag

568:    function excessivelySafeCall(	// @audit-issue missing `@return` tag
```
[15](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L15-L15), [39](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L39-L39), [388](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L388-L388), [394](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L394-L394), [568](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L568-L568), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Function declarations should have `@notice` tag
The `@notice` tag in NatSpec comments is used to provide important explanations to end users about what a function does. It appears that this contract's function declarations are missing `@notice` tags in their NatSpec annotations.

The absence of `@notice` tags reduces the contract's transparency and could lead to misunderstandings about a function's purpose and behavior.  [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

26:    constructor(	// @audit-issue missing `@notice` tag

61:    function owner() public override returns (address) {	// @audit-issue missing `@notice` tag

65:    function doOperation(	// @audit-issue missing `@notice` tag

76:    function _sweepEbtc() private {	// @audit-issue missing `@notice` tag

88:    function _sweepStEth() private {	// @audit-issue missing `@notice` tag

100:    function _getAdjustCdpParams(	// @audit-issue missing `@notice` tag

132:    function _adjustCdpOperation(	// @audit-issue missing `@notice` tag

165:    function _openCdpOperation(	// @audit-issue missing `@notice` tag

200:    function _closeCdpOperation(	// @audit-issue missing `@notice` tag

232:    function _getSwapOperations(	// @audit-issue missing `@notice` tag

249:    function _getSwapChecks(address tokenToCheck, uint256 expectedMinOut) 	// @audit-issue missing `@notice` tag

257:    function _getPostCheckParams(	// @audit-issue missing `@notice` tag

276:    function _openCdpForCallback(bytes memory data) internal override {	// @audit-issue missing `@notice` tag

294:    function _adjustCdpCallback(bytes memory data) internal override {	// @audit-issue missing `@notice` tag
```
[26](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L26-L26), [61](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L61-L61), [65](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L65-L65), [76](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L76-L76), [88](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L88-L88), [100](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L100-L100), [132](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L132-L132), [165](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L165-L165), [200](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L200-L200), [232](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L232-L232), [249](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L249-L249), [257](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L257-L257), [276](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L276-L276), [294](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L294-L294), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

21:    constructor(	// @audit-issue missing `@notice` tag

34:    function openCdpWithEth(	// @audit-issue missing `@notice` tag

77:    function openCdpWithWstEth(	// @audit-issue missing `@notice` tag

112:    receive() external payable {	// @audit-issue missing `@notice` tag

128:    function openCdpWithWrappedEth(	// @audit-issue missing `@notice` tag

171:    function openCdp(	// @audit-issue missing `@notice` tag

205:    function _openCdp(	// @audit-issue missing `@notice` tag

255:    function closeCdp(	// @audit-issue missing `@notice` tag

268:    function closeCdpForWstETH(	// @audit-issue missing `@notice` tag

276:    function _closeCdp(	// @audit-issue missing `@notice` tag

310:    function _requireNonZeroAdjustment(	// @audit-issue missing `@notice` tag

321:    function _requireSingularMarginChange(	// @audit-issue missing `@notice` tag

403:    function _adjustCdp(	// @audit-issue missing `@notice` tag
```
[21](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L21-L21), [34](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L34-L34), [77](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L77-L77), [112](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L112-L112), [128](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L128-L128), [171](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L171-L171), [205](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L205-L205), [255](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L255-L255), [268](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L268-L268), [276](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L276-L276), [310](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L310-L310), [321](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L321-L321), [403](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L403-L403), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

14:    function MIN_CHANGE() external view returns (uint256);	// @audit-issue missing `@notice` tag

27:    constructor(address _borrowerOperations, IERC20 _wstEth, IERC20 _wEth, IStETH _stEth) {	// @audit-issue missing `@notice` tag

34:    function _depositRawEthIntoLido(uint256 _initialETH) internal returns (uint256) {	// @audit-issue missing `@notice` tag

43:    function _convertWrappedEthToStETH(uint256 _initialWETH) internal returns (uint256) {	// @audit-issue missing `@notice` tag

54:    function _convertRawEthToStETH(uint256 _initialETH) internal returns (uint256) {	// @audit-issue missing `@notice` tag

59:    function _convertWstEthToStETH(uint256 _initialWstETH) internal returns (uint256) {	// @audit-issue missing `@notice` tag

72:    function _transferStEthToCaller(	// @audit-issue missing `@notice` tag

107:    function _transferInitialStETHFromCaller(uint256 _initialStETH) internal returns (uint256) {	// @audit-issue missing `@notice` tag

115:    function _permitPositionManagerApproval(	// @audit-issue missing `@notice` tag

135:    function _requireZeroOrMinAdjustment(uint256 _change) internal view {	// @audit-issue missing `@notice` tag

142:    function _requireAtLeastMinNetStEthBalance(uint256 _stEthBalance) internal pure {	// @audit-issue missing `@notice` tag

149:    function _getOwnerAddress(bytes32 cdpId) internal pure returns (address) {	// @audit-issue missing `@notice` tag
```
[14](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L14-L14), [27](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L27-L27), [34](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L34-L34), [43](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L43-L43), [54](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L54-L54), [59](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L59-L59), [72](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L72-L72), [107](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L107-L107), [115](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L115-L115), [135](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L135-L135), [142](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L142-L142), [149](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L149-L149), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

15:    function Cdps(bytes32) external view returns (ICdpManagerData.Cdp memory);	// @audit-issue missing `@notice` tag

39:    function owner() public virtual returns (address) {	// @audit-issue missing `@notice` tag

43:    function _assertOwner() internal {	// @audit-issue missing `@notice` tag

51:    constructor(	// @audit-issue missing `@notice` tag

277:    function _doCheckValueType(CheckValueAndType memory check, uint256 valueToCheck) internal {	// @audit-issue missing `@notice` tag

327:    function _handleOperation(LeverageMacroOperation memory operation) internal {	// @audit-issue missing `@notice` tag

432:    function _doSwaps(SwapOperation[] memory swapData) internal {	// @audit-issue missing `@notice` tag

448:    function _doSwap(SwapOperation memory swapData) internal {	// @audit-issue missing `@notice` tag

485:    function _doSwapChecks(SwapCheck[] memory swapChecks) internal {	// @audit-issue missing `@notice` tag

500:    function _ensureNotSystem(address addy) internal {	// @audit-issue missing `@notice` tag

510:    function _openCdpCallback(bytes memory data) internal virtual {	// @audit-issue missing `@notice` tag

524:    function _openCdpForCallback(bytes memory data) internal virtual {	// @audit-issue missing `@notice` tag

540:    function _closeCdpCallback(bytes memory data) internal virtual {	// @audit-issue missing `@notice` tag

548:    function _adjustCdpCallback(bytes memory data) internal virtual {	// @audit-issue missing `@notice` tag

562:    function _claimSurplusCallback() internal virtual {	// @audit-issue missing `@notice` tag
```
[15](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L15-L15), [39](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L39-L39), [43](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L43-L43), [51](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L51-L51), [277](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L277-L277), [327](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L327-L327), [432](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L432-L432), [448](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L448-L448), [485](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L485-L485), [500](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L500-L500), [510](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L510-L510), [524](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L524-L524), [540](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L540-L540), [548](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L548-L548), [562](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L562-L562), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Function declarations should have `@dev` tag
Some functions have an incomplete NatSpec: add a `@dev` notation to describe the function to improve the code documentation.

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

26:    constructor(	// @audit-issue missing `@dev` tag

61:    function owner() public override returns (address) {	// @audit-issue missing `@dev` tag

65:    function doOperation(	// @audit-issue missing `@dev` tag

76:    function _sweepEbtc() private {	// @audit-issue missing `@dev` tag

88:    function _sweepStEth() private {	// @audit-issue missing `@dev` tag

100:    function _getAdjustCdpParams(	// @audit-issue missing `@dev` tag

132:    function _adjustCdpOperation(	// @audit-issue missing `@dev` tag

165:    function _openCdpOperation(	// @audit-issue missing `@dev` tag

200:    function _closeCdpOperation(	// @audit-issue missing `@dev` tag

232:    function _getSwapOperations(	// @audit-issue missing `@dev` tag

249:    function _getSwapChecks(address tokenToCheck, uint256 expectedMinOut) 	// @audit-issue missing `@dev` tag

257:    function _getPostCheckParams(	// @audit-issue missing `@dev` tag

276:    function _openCdpForCallback(bytes memory data) internal override {	// @audit-issue missing `@dev` tag

294:    function _adjustCdpCallback(bytes memory data) internal override {	// @audit-issue missing `@dev` tag
```
[26](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L26-L26), [61](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L61-L61), [65](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L65-L65), [76](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L76-L76), [88](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L88-L88), [100](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L100-L100), [132](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L132-L132), [165](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L165-L165), [200](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L200-L200), [232](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L232-L232), [249](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L249-L249), [257](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L257-L257), [276](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L276-L276), [294](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L294-L294), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

21:    constructor(	// @audit-issue missing `@dev` tag

205:    function _openCdp(	// @audit-issue missing `@dev` tag

276:    function _closeCdp(	// @audit-issue missing `@dev` tag

310:    function _requireNonZeroAdjustment(	// @audit-issue missing `@dev` tag

321:    function _requireSingularMarginChange(	// @audit-issue missing `@dev` tag

336:    function adjustCdpWithEth(	// @audit-issue missing `@dev` tag

354:    function adjustCdpWithWstEth(	// @audit-issue missing `@dev` tag

372:    function adjustCdpWithWrappedEth(	// @audit-issue missing `@dev` tag

390:    function adjustCdp(	// @audit-issue missing `@dev` tag

403:    function _adjustCdp(	// @audit-issue missing `@dev` tag
```
[21](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L21-L21), [205](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L205-L205), [276](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L276-L276), [310](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L310-L310), [321](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L321-L321), [336](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L336-L336), [354](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L354-L354), [372](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L372-L372), [390](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L390-L390), [403](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L403-L403), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

14:    function MIN_CHANGE() external view returns (uint256);	// @audit-issue missing `@dev` tag

27:    constructor(address _borrowerOperations, IERC20 _wstEth, IERC20 _wEth, IStETH _stEth) {	// @audit-issue missing `@dev` tag

34:    function _depositRawEthIntoLido(uint256 _initialETH) internal returns (uint256) {	// @audit-issue missing `@dev` tag

43:    function _convertWrappedEthToStETH(uint256 _initialWETH) internal returns (uint256) {	// @audit-issue missing `@dev` tag

54:    function _convertRawEthToStETH(uint256 _initialETH) internal returns (uint256) {	// @audit-issue missing `@dev` tag

59:    function _convertWstEthToStETH(uint256 _initialWstETH) internal returns (uint256) {	// @audit-issue missing `@dev` tag

72:    function _transferStEthToCaller(	// @audit-issue missing `@dev` tag

107:    function _transferInitialStETHFromCaller(uint256 _initialStETH) internal returns (uint256) {	// @audit-issue missing `@dev` tag

115:    function _permitPositionManagerApproval(	// @audit-issue missing `@dev` tag

135:    function _requireZeroOrMinAdjustment(uint256 _change) internal view {	// @audit-issue missing `@dev` tag

142:    function _requireAtLeastMinNetStEthBalance(uint256 _stEthBalance) internal pure {	// @audit-issue missing `@dev` tag

149:    function _getOwnerAddress(bytes32 cdpId) internal pure returns (address) {	// @audit-issue missing `@dev` tag
```
[14](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L14-L14), [27](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L27-L27), [34](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L34-L34), [43](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L43-L43), [54](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L54-L54), [59](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L59-L59), [72](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L72-L72), [107](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L107-L107), [115](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L115-L115), [135](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L135-L135), [142](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L142-L142), [149](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L149-L149), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

15:    function Cdps(bytes32) external view returns (ICdpManagerData.Cdp memory);	// @audit-issue missing `@dev` tag

39:    function owner() public virtual returns (address) {	// @audit-issue missing `@dev` tag

43:    function _assertOwner() internal {	// @audit-issue missing `@dev` tag

51:    constructor(	// @audit-issue missing `@dev` tag

119:    function doOperation(	// @audit-issue missing `@dev` tag

247:    function sweepToCaller() public {	// @audit-issue missing `@dev` tag

388:    function decodeFLData(bytes calldata data) public view returns (LeverageMacroOperation memory) {	// @audit-issue missing `@dev` tag

394:    function onFlashLoan(	// @audit-issue missing `@dev` tag

524:    function _openCdpForCallback(bytes memory data) internal virtual {	// @audit-issue missing `@dev` tag

562:    function _claimSurplusCallback() internal virtual {	// @audit-issue missing `@dev` tag
```
[15](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L15-L15), [39](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L39-L39), [43](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L43-L43), [51](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L51-L51), [119](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L119-L119), [247](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L247-L247), [388](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L388-L388), [394](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L394-L394), [524](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L524-L524), [562](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L562-L562), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Function/Constructor `@param` tag is missing
Natural Specification (NatSpec) comments are crucial for understanding the role of function arguments in your Solidity code. Including @param tags will not only improve your code's readability but also its maintainability by clearly defining each argument's purpose. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./ebtc-zap-router/src/LeverageZapRouterBase.sol

26:    constructor(	// @audit-issue missing `@param` tag

65:    function doOperation(	// @audit-issue missing `@param` tag

100:    function _getAdjustCdpParams(	// @audit-issue missing `@param` tag

132:    function _adjustCdpOperation(	// @audit-issue missing `@param` tag

165:    function _openCdpOperation(	// @audit-issue missing `@param` tag

200:    function _closeCdpOperation(	// @audit-issue missing `@param` tag

232:    function _getSwapOperations(	// @audit-issue missing `@param` tag

249:    function _getSwapChecks(address tokenToCheck, uint256 expectedMinOut) 	// @audit-issue missing `@param` tag

257:    function _getPostCheckParams(	// @audit-issue missing `@param` tag

276:    function _openCdpForCallback(bytes memory data) internal override {	// @audit-issue missing `@param` tag

294:    function _adjustCdpCallback(bytes memory data) internal override {	// @audit-issue missing `@param` tag
```
[26](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L26-L26), [65](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L65-L65), [100](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L100-L100), [132](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L132-L132), [165](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L165-L165), [200](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L200-L200), [232](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L232-L232), [249](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L249-L249), [257](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L257-L257), [276](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L276-L276), [294](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/LeverageZapRouterBase.sol#L294-L294), 


```solidity
Path: ./ebtc-zap-router/src/EbtcLeverageZapRouter.sol

21:    constructor(	// @audit-issue missing `@param` tag

205:    function _openCdp(	// @audit-issue missing `@param` tag

276:    function _closeCdp(	// @audit-issue missing `@param` tag

310:    function _requireNonZeroAdjustment(	// @audit-issue missing `@param` tag

321:    function _requireSingularMarginChange(	// @audit-issue missing `@param` tag

403:    function _adjustCdp(	// @audit-issue missing `@param` tag
```
[21](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L21-L21), [205](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L205-L205), [276](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L276-L276), [310](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L310-L310), [321](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L321-L321), [403](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L403-L403), 


```solidity
Path: ./ebtc-zap-router/src/ZapRouterBase.sol

27:    constructor(address _borrowerOperations, IERC20 _wstEth, IERC20 _wEth, IStETH _stEth) {	// @audit-issue missing `@param` tag

34:    function _depositRawEthIntoLido(uint256 _initialETH) internal returns (uint256) {	// @audit-issue missing `@param` tag

43:    function _convertWrappedEthToStETH(uint256 _initialWETH) internal returns (uint256) {	// @audit-issue missing `@param` tag

54:    function _convertRawEthToStETH(uint256 _initialETH) internal returns (uint256) {	// @audit-issue missing `@param` tag

59:    function _convertWstEthToStETH(uint256 _initialWstETH) internal returns (uint256) {	// @audit-issue missing `@param` tag

72:    function _transferStEthToCaller(	// @audit-issue missing `@param` tag

107:    function _transferInitialStETHFromCaller(uint256 _initialStETH) internal returns (uint256) {	// @audit-issue missing `@param` tag

115:    function _permitPositionManagerApproval(	// @audit-issue missing `@param` tag

135:    function _requireZeroOrMinAdjustment(uint256 _change) internal view {	// @audit-issue missing `@param` tag

142:    function _requireAtLeastMinNetStEthBalance(uint256 _stEthBalance) internal pure {	// @audit-issue missing `@param` tag

149:    function _getOwnerAddress(bytes32 cdpId) internal pure returns (address) {	// @audit-issue missing `@param` tag
```
[27](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L27-L27), [34](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L34-L34), [43](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L43-L43), [54](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L54-L54), [59](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L59-L59), [72](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L72-L72), [107](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L107-L107), [115](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L115-L115), [135](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L135-L135), [142](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L142-L142), [149](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-zap-router/src/ZapRouterBase.sol#L149-L149), 


```solidity
Path: ./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

15:    function Cdps(bytes32) external view returns (ICdpManagerData.Cdp memory);	// @audit-issue missing `@param` tag

51:    constructor(	// @audit-issue missing `@param` tag

119:    function doOperation(	// @audit-issue missing `@param` tag

267:    function sweepToken(address token, uint256 amount) public {	// @audit-issue missing `@param` tag

277:    function _doCheckValueType(CheckValueAndType memory check, uint256 valueToCheck) internal {	// @audit-issue missing `@param` tag

327:    function _handleOperation(LeverageMacroOperation memory operation) internal {	// @audit-issue missing `@param` tag

388:    function decodeFLData(bytes calldata data) public view returns (LeverageMacroOperation memory) {	// @audit-issue missing `@param` tag

394:    function onFlashLoan(	// @audit-issue missing `@param` tag

432:    function _doSwaps(SwapOperation[] memory swapData) internal {	// @audit-issue missing `@param` tag

448:    function _doSwap(SwapOperation memory swapData) internal {	// @audit-issue missing `@param` tag

485:    function _doSwapChecks(SwapCheck[] memory swapChecks) internal {	// @audit-issue missing `@param` tag

500:    function _ensureNotSystem(address addy) internal {	// @audit-issue missing `@param` tag

510:    function _openCdpCallback(bytes memory data) internal virtual {	// @audit-issue missing `@param` tag

524:    function _openCdpForCallback(bytes memory data) internal virtual {	// @audit-issue missing `@param` tag

540:    function _closeCdpCallback(bytes memory data) internal virtual {	// @audit-issue missing `@param` tag

548:    function _adjustCdpCallback(bytes memory data) internal virtual {	// @audit-issue missing `@param` tag

568:    function excessivelySafeCall(	// @audit-issue missing `@param` tag
```
[15](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L15-L15), [51](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L51-L51), [119](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L119-L119), [267](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L267-L267), [277](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L277-L277), [327](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L327-L327), [388](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L388-L388), [394](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L394-L394), [432](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L432-L432), [448](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L448-L448), [485](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L485-L485), [500](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L500-L500), [510](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L510-L510), [524](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L524-L524), [540](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L540-L540), [548](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L548-L548), [568](https://github.com/code-423n4/2024-06-badger/blob/61df0ce381d5e1f10191257aaa304fac4776ad33/./ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L568-L568), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).
