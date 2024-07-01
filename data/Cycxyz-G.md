file ZapRouterBase.sol line 43
```
function _convertWrappedEthToStETH(uint256 _initialWETH) internal returns (uint256) {
        uint256 _wETHBalBefore = wrappedEth.balanceOf(address(this));
        wrappedEth.transferFrom(msg.sender, address(this), _initialWETH);
        uint256 _wETHReiceived = wrappedEth.balanceOf(address(this)) - _wETHBalBefore;

        uint256 _rawETHBalBefore = address(this).balance;
        IWrappedETH(address(wrappedEth)).withdraw(_wETHReiceived);
        uint256 _rawETHConverted = address(this).balance - _rawETHBalBefore;
        return _depositRawEthIntoLido(_rawETHConverted);
    }
```

no need to save balance before transfer, you can be sure you received requested amount. Otherwise transaction will be reverted. It's actual for _wETHBalanceBefore and _rawETHBalBefore