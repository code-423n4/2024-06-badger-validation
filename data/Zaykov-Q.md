**Typo in `_convertWstEthToStETH`**

https://github.com/code-423n4/2024-06-badger/blob/22de3b03bbb2fc4f04c362d29a2cfb4211235884/ebtc-zap-router/src/ZapRouterBase.sol#L59-L70

There is a typo. It should be `_stETHReceived`
```
   function _convertWstEthToStETH(uint256 _initialWstETH) internal returns (uint256) {
        require(
            wstEth.transferFrom(msg.sender, address(this), _initialWstETH),
            "EbtcZapRouter: transfer wstETH failure!"
        );

        uint256 _stETHBalBefore = stEth.balanceOf(address(this));
        IWstETH(address(wstEth)).unwrap(_initialWstETH);
        uint256 _stETHReiceived = stEth.balanceOf(address(this)) - _stETHBalBefore;

        return _stETHReiceived;
    }
```