## Use assembly in place of abi.decode to extract calldata values more efficiently

Instead of using abi.decode, leveraging assembly allows for direct extraction of necessary calldata values. This approach optimizes gas usage by skipping unnecessary decoding operations.

## Lines of code 
https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/LeverageZapRouterBase.sol#L276-L314

## Refactored Functions Using Assembly
```
function _openCdpForCallback(bytes memory data) internal override {
    if (zapFeeBPS > 0) {
        require(data.length == 160, "Invalid data length");
 
        address _upperHint;
        address _lowerHint;
        uint256 eBTCToMint;
        uint256 stETHToDeposit;
        address borrower;
        
        assembly {
            eBTCToMint := mload(add(data, 32))
            _upperHint := mload(add(data, 64))
            _lowerHint := mload(add(data, 96))
            stETHToDeposit := mload(add(data, 128))
            borrower := mload(add(data, 160))
        }

        bytes32 _cdpId = borrowerOperations.openCdpFor(
            eBTCToMint,
            _upperHint,
            _lowerHint,
            stETHToDeposit,
            borrower
        );

        IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, eBTCToMint * zapFeeBPS / BPS);
    } else {
        super._openCdpForCallback(data);
    }
}
```

```
function _adjustCdpCallback(bytes memory data) internal override {
    if (zapFeeBPS > 0) {
        require(data.length == 224, "Invalid data length");
        uint256 _cdpId;
        uint256 _stEthBalanceDecrease;
        uint256 _EBTCChange;
        bool _isDebtIncrease;
        address _upperHint;
        address _lowerHint;
        uint256 _stEthBalanceIncrease;

        assembly {
            _cdpId := mload(add(data, 32))
            _stEthBalanceDecrease := mload(add(data, 64))
            _EBTCChange := mload(add(data, 96))
            _isDebtIncrease := mload(add(data, 128))
            _upperHint := mload(add(data, 160))
            _lowerHint := mload(add(data, 192))
            _stEthBalanceIncrease := mload(add(data, 224))
        }

        borrowerOperations.adjustCdpWithColl(
            _cdpId,
            _stEthBalanceDecrease,
            _EBTCChange,
            _isDebtIncrease,
            _upperHint,
            _lowerHint,
            _stEthBalanceIncrease
        );

        if (_isDebtIncrease) {
            IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, _EBTCChange * zapFeeBPS / BPS);
        }
    } else {
        super._adjustCdpCallback(data);
    }
}
```
