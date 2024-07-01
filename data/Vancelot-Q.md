# [L-01] The stETH balance should be tracked using shares

## Summary

The intended way of transferring `stETH` by Lido is via the [transferShares](https://docs.lido.fi/contracts/lido#transfershares) function, however, in order for the shares to be tracked accordingly, the correct function should be used. Judging by the NatSpec throughout the protocol, the developers are aware of the [1-2 wei issue](https://docs.lido.fi/guides/lido-tokens-integration-guide/#1-2-wei-corner-case) and have handled it well, but the [ZapRouterBase](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol) contract has several instances where the balance of the contract is checked using `balanceOf` whereas the [sharesOf](https://docs.lido.fi/contracts/lido#sharesof) function should be used.

Instances of using balanceOf:

- [__depositRawEthIntoLido](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol#L36-L39)
- [_convertWstEthToStETH](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol#L65-L67)
- [_transferInitialStETHFromCaller](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol#L109-L111)

## Recommendations

Use [sharesOf](https://docs.lido.fi/contracts/lido#sharesof) instead of balanceOf to check the actual balance of `stETH`.

# [L-02] Usage of low-level call instead of submit

## Summary

When transferring native/raw Eth into Lido’s pool, in the [ZapRouterBase](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol) contract, the function [_depositRawEthIntoLido](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol#L34-L41) is used, within which a `low-level call` is executed. That same call would be enough to handle the transfer of funds, as the fallback function stakes the deposited Eth for the `msg.sender`. The issue is that the protocol is missing out on potential referral rewards by not using the [submit](https://docs.lido.fi/contracts/lido#submit-1) function instead of directly transferring the raw Ether.

As the contract should not hold any funds, a new address should be created, in which all of the referral rewards will be collected.  

## Recommendations

Consider making the following changes in the function [_depositRawEthIntoLido](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol#L34-L41)

```diff
     function _depositRawEthIntoLido(uint256 _initialETH) internal returns (uint256) {
         // check before-after balances for 1-wei corner case
         uint256 _balBefore = stEth.balanceOf(address(this));
         // TODO call submit() with a referral?
-        payable(address(stEth)).call{value: _initialETH}("");
+        payable(address(stEth)).submit{value: _initialETH}(address(this));
         uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;
         return _deposit;
     }
```