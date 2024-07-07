## Title - `_transferStEthToCaller` will still run if `_stEthVal` is 0, can cause wastage of Gas
## Vulnerability Details

In the function `_transferStEthToCaller` of `ZapRouterBase` there is no check present to ensure if `_stEthVal` can be set to 0.
If we enter `_useWstETH=false` and `_stEthVal=0` the `else` part will run and `stEth.transfer(msg.sender, _stEthVal)` will transfer `0` value. Which can be wastage of gas.

## PoC

```solidity
    function _transferStEthToCaller(
        bytes32 _cdpId,
        EthVariantZapOperationType _operationType,
        bool _useWstETH,
        uint256 _stEthVal
    ) internal {
        if (_useWstETH) {
            // return wrapped version(WstETH)
            uint256 _wstETHVal = IWstETH(address(wstEth)).wrap(_stEthVal);
            emit ZapOperationEthVariant(
                _cdpId,
                _operationType,
                false,
                address(wstEth),
                _wstETHVal,
                _stEthVal,
                msg.sender
            );

            wstEth.transfer(msg.sender, _wstETHVal);
        } else {
            // return original collateral(stETH)
            emit ZapOperationEthVariant(
                _cdpId,
                _operationType,
                false,
                address(stEth),
                _stEthVal,
                _stEthVal,
                msg.sender
            );
            stEth.transfer(msg.sender, _stEthVal); //@audit- if _useWstETH= false,_stEthVal=0 else part will run lead to gas issues 
            //but no transfer will be done
        }
    }
   ```
## Impact

Wastage of gas fees

## Code snippet

[Code link](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/ZapRouterBase.sol#L72-L105)

## Tools Used 

Manual Review

## Recommended Mitigation

Use checks to ensure  `_stEthVal` can't be 0