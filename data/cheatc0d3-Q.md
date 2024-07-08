# QA Report: Badger eBTC Zap Router

## L-01: zapFeeReceiver cannot be updated after contract deployment which can be problematic

### Issue
The LeverageZapRouterBase contract sets the zapFeeReceiver address as an immutable variable during contract deployment. This design choice prevents the fee recipient address from being updated, which could lead to issues if the address needs to be changed due to security concerns, operational requirements, or governance decisions.

The contract should allow authorized parties to update the zapFeeReceiver address when necessary.

## Code

https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol#L52

```solidity
address public immutable zapFeeReceiver;

```

### Impact
1. If the zapFeeReceiver address is compromised, there's no way to redirect fees to a new, secure address.
2. The protocol loses flexibility in adjusting its fee collection strategy or beneficiary.
3. If the original address becomes invalid or inaccessible, fees could be permanently lost.

### Recommendation
Implement a setter function with appropriate access control

## L-02: Implementing a pausable fee mechanism that allows fees to be temporarily disabled if needed

### Issue

The LeverageZapRouterBase contract lacks a mechanism to temporarily disable or pause fee collection. This inflexibility could be problematic in certain scenarios where fee suspension might be necessary or beneficial.

Current Behavior:
- Fees are always collected when zapFeeBPS > 0
- There is no way to temporarily stop fee collection without setting zapFeeBPS to 0, which requires a new contract deployment due to its immutable nature

Expected Behavior:
- The contract should have a mechanism to temporarily pause fee collection without changing the zapFeeBPS value
- This pause functionality should be controllable by authorized parties (e.g., contract owner or governance)

### Code:

https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol#L276

```solidity
function _openCdpForCallback(bytes memory data) internal override {
    if (zapFeeBPS > 0) {
        OpenCdpForOperation memory flData = abi.decode(data, (OpenCdpForOperation));

        bytes32 _cdpId = borrowerOperations.openCdpFor(
            flData.eBTCToMint,
            flData._upperHint,
            flData._lowerHint,
            flData.stETHToDeposit,
            flData.borrower
        );

        IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData.eBTCToMint * zapFeeBPS / BPS);
    } else {
        super._openCdpForCallback(data);
    }
}
```

### Impact
- Inability to quickly respond to market conditions or emergencies that might require temporary fee suspension
- Reduced flexibility in protocol management and potential user experience issues

### Recommendation
Implement a pausable mechanism for fee collection.

## L-03: Infinite approvals can be risky if the approved contracts are compromised

### Issue
The contract LeverageZapRouterBase includes infinite token approvals in its constructor. These approvals grant unlimited spending rights to several external contracts, which could lead to a complete loss of funds if any of the approved contracts are compromised or contain vulnerabilities.


https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol#L56


```solidity
stEth.approve(address(borrowerOperations), type(uint256).max);
stEth.approve(address(wstEth), type(uint256).max);
```

While this is common practice to reduce gas costs for users, it can be a security risk if the approved contracts have vulnerabilities or are compromised. It's generally safer to approve only the amount needed for each transaction.

### Recommendation
Instead of setting infinite approvals in the constructor, implement a system where approvals are set for the specific amount needed just before each transaction that requires it.

## L-04: Lack of indexed parameters in events, making off-chain filtering less efficient

### Issue
The EbtcLeverageZapRouter contract defines several events, but none of the event parameters are marked as `indexed`. Indexing important parameters in events allows for more efficient off-chain filtering and querying of event logs.

### Impact
The lack of indexed parameters makes it more difficult and less efficient for off-chain services to filter and query specific events. This can lead to increased processing time and resource usage when analyzing event logs, especially as the number of events grows over time.

### Recommendation
Consider adding the `indexed` keyword to important parameters in the events. For example, the `ZapOperationEthVariant` event could be modified as follows:

https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L100

```solidity
event ZapOperationEthVariant(
    bytes32 indexed cdpId,
    EthVariantZapOperationType indexed operationType,
    bool isIncrease,
    address asset,
    uint256 amount,
    uint256 collateralValue,
    address indexed user
);
```

## L-05: Lack of Time-Lock on Owner's Token Sweep Functions Enables Contract's Assets to be drained

### Issue

The LeverageMacroBase contract allows the owner to immediately sweep any tokens held by the contract, including eBTC, stETH, and any arbitrary ERC20 token, without any time delay. This lack of a time-lock mechanism could be problematic if the owner's private key is compromised or if there's a malicious owner.

### Code:

1. General sweep function:

https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L247

```solidity
function sweepToCaller() public {
    _assertOwner();
    uint256 ebtcBal = ebtcToken.balanceOf(address(this));
    uint256 collateralBal = stETH.sharesOf(address(this));

    if (ebtcBal > 0) {
        ebtcToken.transfer(msg.sender, ebtcBal);
    }

    if (collateralBal > 0) {
        stETH.transferShares(msg.sender, collateralBal);
    }
}
```

2. Arbitrary token sweep function:

https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L267

```solidity
function sweepToken(address token, uint256 amount) public {
    _assertOwner();
    IERC20(token).safeTransfer(msg.sender, amount);
}
```

### Impact
If an attacker gains control of the owner's account, they could immediately drain all funds from the contract, including any ERC20 token. This poses a significant risk to any user funds that may be temporarily held by the contract during macro operations or any other tokens that might accidentally be sent to the contract.

### Recommendation
Implement a time-lock mechanism for both sweeping functionalities. This could involve:

1. Adding a waiting period between initiating a sweep and executing it.
2. Emitting an event when a sweep is initiated.
3. Allowing the sweep to be canceled during the waiting period.


## L-06: Use ownable2step ownership standard

### Issue

The contract LeverageZapRouterBase implements a custom ownership mechanism using an immutable `theOwner` variable and an `owner()` function. However, this implementation lacks the security benefits of more robust ownership patterns like OpenZeppelin's Ownable2Step.

The current implementation:
1. Sets the owner in the constructor:

https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol#L49

```solidity
theOwner = params.owner;
```
2. Provides a simple `owner()` function:

https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol#L61

```solidity
function owner() public override returns (address) {
    return theOwner;
}
```


### Recommendation
Implement the Ownable2Step pattern from OpenZeppelin, which provides:


### L-07: No pause mechanism for emergencies

### Issue
The LeverageZapRouterBase contract, which handles critical financial operations including flash loans and CDP (Collateralized Debt Position) management, lacks a pause mechanism. This omission could prevent quick response to potential security threats or bugs, potentially leading to significant financial losses.

https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol

### Impact
Without a pause function:
1. The contract cannot be halted in case of detected vulnerabilities or ongoing attacks.
2. Developers lack the ability to temporarily stop operations for critical updates or bug fixes.
3. User funds could be at risk if a security issue is identified but can't be immediately mitigated.


### Recommendation
Implement a pause mechanism using OpenZeppelin's Pausable contract. This would allow authorized parties to halt critical functions in emergencies. 