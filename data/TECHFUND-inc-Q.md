
## Lines of Code:
https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/LeverageZapRouterBase.sol#L257C1-L275C1

## Description : 

The `LeverageZapRouterBase::_getPostCheckParams` function is used to provide a way to create and return the necessary parameters for verifying that a leverage operation has been executed correctly. However, the function is using the fixed operators (`Operator.equal` for debt and `Operator.gte` for collateral) will not always be appropriate for the checks being performed. If different scenarios or operations require other comparison operators, the function as currently implemented may not handle them correctly.
By hardcoding the operators (`Operator.equal` and `Operator.gte`), the function assumes these checks are always correct. This limits the flexibility to adapt to different business logic requirements that might arise.

Here's a possible situation:
If collateral is being withdrawn, the check might need to ensure the remaining collateral is `Operator.lte` a certain threshold, rather than always being `Operator.gte` .


### Tools Used :
Manual Audit

## Recommended Mitigation : 
To handle different scenarios, the function can be modified to accept operators as parameters. This allows the caller to specify the appropriate comparison operator for each check.
