## Unused return storage variables


Unused storage variables in contracts use up storage slots and increase contract size and gas usage at deployment or execution time .

The `_openCdp()` function in the EbtcLeverageZapRouter.sol contract has an unused parameter `uint256 _stEthMarginAmount.` This function is called by four other functions: `openCdpWithEth(), openCdpWithWstEth(), openCdpWithWrappedEth(), and openCdp()`.

## Proof of Concept

```
https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L51

https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L94

https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L145

https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L188

```

```
 cdpId = _openCdp(
            _debt,
            _upperHint,
            _lowerHint,
            _stEthLoanAmount,
            _collVal,               @audit-issue unused variable 
            _stEthDepositAmount,
            _positionManagerPermit,
            _tradeData
        );



 function _openCdp(
        uint256 _debt,
        bytes32 _upperHint,
        bytes32 _lowerHint,
        uint256 _stEthLoanAmount,
        uint256 _stEthMarginAmount,   @audit-issue unused variable 
        uint256 _stEthDepositAmount,
        bytes calldata _positionManagerPermit,
        TradeData calldata _tradeData
    ) internal nonReentrant returns (bytes32 cdpId) {
...
}
```




## Recommended Mitigation Steps

* Remove unused variables to reduce the deployment costs.
