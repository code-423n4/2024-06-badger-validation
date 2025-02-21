**File ZapRouterBase.sol line 43**
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

**File ZapRouterBase.sol line 36**
```
function _depositRawEthIntoLido(uint256 _initialETH) internal returns (uint256) {
        // check before-after balances for 1-wei corner case
        uint256 _balBefore = stEth.balanceOf(address(this));
        // TODO call submit() with a referral?
        payable(address(stEth)).call{value: _initialETH}("");
        uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;
        return _deposit;
}
```
no need to save _balBefore, you can be sure you'll receive amount of shares(WSTETH) appropriate to _initialETH value. You need to return _initiailETH instead of subtraction result and work with shares(WSTETH) not with calculated from shares values(STETH)