QA1. L492 has the wrong comparison operator, as a result, even when ```swapChecks[i].tokenToCheck).balanceOf(address(this) = swapChecks[i].expectedMinOut```, it will still fail since the compariosn operator is ```>``` instead of ```>=```. 

[https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L485-L497](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L485-L497)

Mitigation: change the comparison operator to ```>=```. 