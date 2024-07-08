# [QA-01] Missing Return Value Check in External Call to `stEth.submit`

## Description
In the `ZapRouterBase` contract, the external call to `stEth.submit` is made without handling the return value. Specifically, the line:
```
payable(address(stEth)).call{value: _initialETH}("");
```
This call does not check if the operation was successful. If the call to `stEth.submit` fails, the contract would not detect the failure, potentially leading to incorrect behavior and security vulnerabilities.
https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/ZapRouterBase.sol#L34-L42

# Recommendation
To ensure the call to `stEth.submit` is successful, modify the code to handle the return value properly. This can be done by checking the boolean value returned by the call function. Here is the recommended change:
```
(bool success, ) = payable(address(stEth)).call{value: _initialETH}("");
require(success, "Deposit to Lido failed");
```
# [QA-02] Error Handling in `_permitPositionManagerApproval`

## Description
In the `ZapRouterBase` contract, the `_permitPositionManagerApproval` function includes a `try...catch` block when calling `borrowerOperations.permitPositionManagerApproval`. This error handling mechanism is designed to mitigate potential issues, such as permit front-running attacks, that could arise during the execution of the permit approval process.
Issue: The catch block is empty. This can lead to silent failures.
https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/ZapRouterBase.sol#L129-L130
## Recommendation
Implement logging within the catch block to provide visibility into any errors encountered during the permit approval process.
```
{} catch {
        emit PermitApprovalFailed(msg.sender, "Permit approval failed");
```

# [QA-03] Typographical Error in `_convertWrappedEthToStETH` Function
There are several typographical errors in the code which affect readability and maintainability. These errors include misspelled variable names and minor inconsistencies that can cause confusion during code review and maintenance.
## First instance:
In _convertWrappedEthToStETH function:
`_wETHReiceived` should be `_wETHReceived`
https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/ZapRouterBase.sol#L46-L47

## Second instance:
In `_convertWstEthToStETH` function:
`_stETHReiceived` should be `_stETHReceived`
https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/ZapRouterBase.sol#L67-L68