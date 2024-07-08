Can't reopen a closed CDP. We can move this judge at the beginning of _adjustCdp.
If anyone who want to reopen a closed Cdp, it will revert ASAP.

```
    function _adjustCdp(
        bytes32 _cdpId,
        AdjustCdpParams memory _params,
        bytes calldata _positionManagerPermit,
        TradeData calldata _tradeData,
        uint256 _zapStEthBalanceBefore
    ) internal nonReentrant {
        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for adjust!");
        _requireZeroOrMinAdjustment(_params.debtChange);
        _requireZeroOrMinAdjustment(_params.stEthBalanceChange);
        _requireZeroOrMinAdjustment(_params.stEthMarginBalance);
        // @audit
        // address _borrower = sortedCdps.getOwnerAddress(_cdpId);
        // _requireBorrowerOrPositionManagerAndUpdateManagerApproval(_borrower);
```
