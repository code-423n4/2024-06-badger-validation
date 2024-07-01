- File *ZapRouterBase.sol*, line 36.
```
function _depositRawEthIntoLido(uint256 _initialETH) internal returns (uint256) {
        // check before-after balances for 1-wei corner case
        uint256 _balBefore = stEth.balanceOf(address(this));
        // TODO call submit() with a referral?
        payable(address(stEth)).call{value: _initialETH}("");
        uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;
        return _deposit;
}
````
 No need to calculate balance before, you can just return _initialEth. Be sure you received number of shares(WSTETH) appropriate to supplied _initialETH value