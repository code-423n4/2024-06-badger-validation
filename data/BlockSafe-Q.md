## Missing Deadline Checks in Swap Operations
[LeverageZapRouterBase.sol#L232-L247](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/LeverageZapRouterBase.sol#L232-L247)
[LeverageMacroBase.sol#L448-L481](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L448-L481)
The `LeverageZapRouterBase` contract facilitates various complex operations involving collateralized debt positions (CDPs), including token swaps via decentralized exchanges (DEX). However, the contract's swap operations do not include deadline checks, which could lead to significant risk exposure due to price slippage or other adverse market conditions.

In the [_getSwapOperations](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/LeverageZapRouterBase.sol#L232-L247) function, swap operations are constructed without any deadline parameter to ensure that the swaps are executed within a specific timeframe:
```solidity
function _getSwapOperations(
    address _tokenIn,
    address _tokenOut,
    TradeData calldata _tradeData
) internal view returns (SwapOperation[] memory swaps) {
    swaps = new SwapOperation ;

    swaps[0].tokenForSwap = _tokenIn;
    swaps[0].addressForApprove = DEX;
    swaps[0].exactApproveAmount = _tradeData.approvalAmount;
    swaps[0].addressForSwap = DEX;
    swaps[0].calldataForSwap = _tradeData.exchangeData;
    if (_tradeData.performSwapChecks) {
        swaps[0].swapChecks = _getSwapChecks(_tokenOut, _tradeData.expectedMinOut);            
    }
}
```

The [SwapOperation](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L301-L309) struct and the [SwapCheck](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L311-L315) struct and the related functions lack a deadline parameter, meaning there is no mechanism to enforce timely execution of the swaps, which is crucial for minimizing risks associated with rapidly changing market conditions.

[_doSwap](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L448-L481) Function

```solidity
function _doSwap(SwapOperation memory swapData) internal {
    // Ensure call is safe
    _ensureNotSystem(swapData.addressForSwap);

    // Exact approve
    IERC20(swapData.tokenForSwap).safeApprove(
        swapData.addressForApprove,
        swapData.exactApproveAmount
    );

    // Call and perform swap
    (bool success, ) = excessivelySafeCall(
        swapData.addressForSwap,
        gasleft(),
        0,
        0,
        swapData.calldataForSwap
    );
    require(success, "Call has failed");

    // Approve back to 0
    IERC20(swapData.tokenForSwap).safeApprove(swapData.addressForApprove, 0);

    // Do the balance checks after the call to the aggregator
    _doSwapChecks(swapData.swapChecks);
}
```
In the `_doSwap` function, the swap operation is performed by calling the target contract specified in `addressForSwap` with the provided calldata in `calldataForSwap`. There is no parameter or logic to enforce a deadline for the swap operation.
## Impact
Without deadline checks, swap operations may be executed under unfavorable market conditions, leading to significant price slippage. This can result in substantial financial losses for users due to the execution of trades at less favorable rates than anticipated.

## Mitigation
Modify the `SwapOperation` struct to include a deadline parameter and then check this parameter in the `_doSwap` function before executing the swap.