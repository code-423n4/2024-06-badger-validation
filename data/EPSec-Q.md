# EPSec Report

This report was created by EPSec.

# Summary

## Issue Summary

| Category | No. of Issues |
| -------- | ------------- |
| Low      | 4             |
| Info     | 0             |
## Lows

## Low-01: Response of flash loan is not checked


`LeverageMacroBase` contains code that performs flash loans using different lenders based on the type of flash loan. However, the code does not verify whether the flash loan operation was successful. While this is currently acceptable because the flash loan functions are designed to either return true or revert, this could pose a risk if the implementation of the flash loan functions changes in the future.

## Impact
If the flash loan implementation changes to return false instead of reverting on failure, the lack of a success check could lead to undetected failures, potentially resulting in incorrect execution of subsequent operations. This can compromise the integrity and reliability of the contract, leading to unexpected behaviors and potential financial losses.

## Proof of Concept
Consider the following code snippet from the smart contract:

```solidity
if (flType == FlashLoanType.eBTC) {
    IERC3156FlashLender(address(borrowerOperations)).flashLoan(
        IERC3156FlashBorrower(address(this)),
        address(ebtcToken),
        borrowAmount,
        abi.encode(operation)
    );
} else if (flType == FlashLoanType.stETH) {
    IERC3156FlashLender(address(activePool)).flashLoan(
        IERC3156FlashBorrower(address(this)),
        address(stETH),
        borrowAmount,
        abi.encode(operation)
    ); // we borrow money
}
```

In this code, the result of the `flashLoan` function calls is not checked. If the flash loan functions are modified to return false on failure instead of reverting, the contract will not be able to detect and handle such failures, leading to potential issues.

## Recommendation
To ensure the robustness and reliability of the contract, it is recommended to add checks to verify the success of the flash loan operations. This can be done by capturing the return value of the `flashLoan` function and reverting the transaction if the return value is false.

### Implementation
Update the flash loan code to include success verification checks:

```solidity
if (flType == FlashLoanType.eBTC) {
    bool success = IERC3156FlashLender(address(borrowerOperations)).flashLoan(
        IERC3156FlashBorrower(address(this)),
        address(ebtcToken),
        borrowAmount,
        abi.encode(operation)
    );
    require(success, "Flash loan for eBTC failed");
} else if (flType == FlashLoanType.stETH) {
    bool success = IERC3156FlashLender(address(activePool)).flashLoan(
        IERC3156FlashBorrower(address(this)),
        address(stETH),
        borrowAmount,
        abi.encode(operation)
    ); // we borrow money
    require(success, "Flash loan for stETH failed");
}
```

This update ensures that if the flash loan operation fails, the transaction is reverted, thereby preventing any further execution and potential issues.

## Low-02: Do swaps iterates could not check correct

The `_doSwaps` function iterates over an array of swap data and calls `_doSwapChecks` to verify if the contract's balance for a specific token exceeds a certain `minAmount`. However, if the same token was used in a previous swap, the require statement might still pass even if the balance does not meet the `minAmount` requirement, potentially leading to inaccurate balance checks and slippage protection issues.

## Impact

If the `_doSwapChecks` function does not correctly verify the contract's balance for a token, subsequent swaps involving the same token may proceed without ensuring that the balance meets the required `minAmount`. This could result in slippage protection not working as expected, leading to unfavorable trading conditions and potential financial losses for users.
## Low-03:  expectedCdpId could be zero


The `doOperation` function in `LeverageMacroBase` may have an uninitialized `expectedCdpId`, leading to potential issues if the `expectedCdpId` is used without proper initialization. This could bypass checks and lead to unintended operations, including the manipulation of a CDP with an ID of zero if such a CDP exists.

## Impact
An uninitialized `expectedCdpId` can result in:
1. **Bypassing Critical Checks:** The function may proceed without proper validation, leading to unintended consequences.
2. **Manipulation of CDP with ID Zero:** If a CDP with ID zero exists, it could be erroneously affected by operations intended for a different CDP.
3. **Security Vulnerabilities:** Attackers could exploit this flaw to manipulate the protocol, leading to potential financial losses and instability.

## Proof of Concept
Consider the following code snippet from the `doOperation` function:

```solidity
function doOperation(
    FlashLoanType flType,
    uint256 borrowAmount,
    LeverageMacroOperation calldata operation,
    PostOperationCheck postCheckType,
    PostCheckParams calldata checkParams
) external virtual {
    _assertOwner();

    // Figure out the expected CDP ID using sortedCdps.toCdpId
    bytes32 expectedCdpId;
    if (
        operation.operationType == OperationType.OpenCdpOperation &&
        postCheckType != PostOperationCheck.none
    ) {
        expectedCdpId = sortedCdps.toCdpId(
            address(this),
            block.number,
            sortedCdps.nextCdpNonce()
        );
    } else if (
        operation.operationType == OperationType.OpenCdpForOperation &&
        postCheckType != PostOperationCheck.none
    ) {
        OpenCdpForOperation memory flData = abi.decode(
            operation.OperationData,
            (OpenCdpForOperation)
        );
        // This is used to support permitPositionManagerApproval
        expectedCdpId = sortedCdps.toCdpId(
            flData.borrower,
            block.number,
            sortedCdps.nextCdpNonce()
        );
    }

    _doOperation(flType, borrowAmount, operation, postCheckType, checkParams, expectedCdpId);
}
```

If neither condition for initializing `expectedCdpId` is met, `expectedCdpId` remains uninitialized, potentially leading to the issues described.

## Recommendation
To prevent these issues, the function should include a revert statement when `expectedCdpId` is not initialized. This ensures that the function does not proceed with an uninitialized or zero `expectedCdpId`.

Add a check to revert the transaction if `expectedCdpId` is not set:

```diff
function doOperation(
    FlashLoanType flType,
    uint256 borrowAmount,
    LeverageMacroOperation calldata operation,
    PostOperationCheck postCheckType,
    PostCheckParams calldata checkParams
) external virtual {
    _assertOwner();

    // Figure out the expected CDP ID using sortedCdps.toCdpId
    bytes32 expectedCdpId;
    if (
        operation.operationType == OperationType.OpenCdpOperation &&
        postCheckType != PostOperationCheck.none
    ) {
        expectedCdpId = sortedCdps.toCdpId(
            address(this),
            block.number,
            sortedCdps.nextCdpNonce()
        );
    } else if (
        operation.operationType == OperationType.OpenCdpForOperation &&
        postCheckType != PostOperationCheck.none
    ) {
        OpenCdpForOperation memory flData = abi.decode(
            operation.OperationData,
            (OpenCdpForOperation)
        );
        // This is used to support permitPositionManagerApproval
        expectedCdpId = sortedCdps.toCdpId(
            flData.borrower,
            block.number,
            sortedCdps.nextCdpNonce()
        );
+    } else {
+        revert("Invalid operation type or post-check type");
+    }

    _doOperation(flType, borrowAmount, operation, postCheckType, checkParams, expectedCdpId);
}
```

This update ensures that the function reverts if `expectedCdpId` is not properly initialized, preventing unintended operations and potential exploits.

## Low-04: Close cdp missing checks

In the `doOperation` function, the post-operation checks currently verify only the status of the CDP after closing a position. However, checks for the debt and collateral of the CDP are not enforced. This oversight can lead to inconsistencies and potential vulnerabilities. This report outlines the necessary code updates to enforce these checks.

## Impact

Without checks on the debt and collateral, the post-operation verification is incomplete, potentially allowing discrepancies in the CDP state that could lead to financial losses or manipulation of the protocol.

## Recommendation

Add checks for debt and collateral alongside the existing status check in the post-operation verification.

### Implementation

Update the `doOperation` function to include debt and collateral checks for both the `cdpStats` and `isClosed` post-check types:

```diff
if (postCheckType == PostOperationCheck.isClosed) {
    ICdpManagerData.Cdp memory cdpInfo = cdpManager.Cdps(checkParams.cdpId);

     _doCheckValueType(checkParams.expectedDebt, cdpInfo.debt);
    _doCheckValueType(checkParams.expectedCollateral, cdpInfo.coll);
    require(
        cdpInfo.status == checkParams.expectedStatus,
        "!LeverageMacroReference: closeCDP status check"
    );
}
```