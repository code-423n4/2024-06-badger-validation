## Title - not handling return values of `transferFrom`

## Vulnerability Details
As per the ERC20 OpenZeppelin docs, `transferFrom` definition is 
`transferFrom(address sender, address recipient, uint256 amount) → bool` 
[ERC20 Openzeppelin](https://docs.openzeppelin.com/contracts/2.x/api/token/erc20)

Hence, it should return a Boolean value, but in all the functions of the return value of `transferFrom` is not handled properly. Contract might assume that all transfers are successful, which might not be the case always. 

## Impact
If we are not checking the return value of transferFrom, we are essentially ignoring whether the transfer was successful or not. It can lead to
a) Silent failures
b) Incorrect state management

## Code snippet

[(Provide links or code snippet of the related finding)](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/EbtcZapRouter.sol#L477)

https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/EbtcZapRouter.sol#L517

https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/ZapRouterBase.sol#L45

https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/ZapRouterBase.sol#L110

## Tools Used 

Manual review

## Recommended Mitigation

Return value of `transferFrom` should always be checked.
E.g.
```solidity
bool success = token.transferFrom(sender, recipient, amount);
require(success, "Transfer failed");
```
