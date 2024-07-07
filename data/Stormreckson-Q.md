


1-  Redundant Code: Unused `openCdpCallback` Function

## Vulnerability Discussion
 The `openCdpCallback` function is defined but never used. Instead, the `openCdpForCallback` function is utilized for all operations related to opening a CDP. The `openCdpForCallback` includes logic for charging fees, which is absent in the `openCdpCallback`. This redundancy in the code not only causes confusion but also increases the potential for future maintenance issues and misunderstandings regarding the contract’s intended functionality.

## Impact
 Redundant code can lead to confusion for developers maintaining the contract, making it harder to understand the codebase.
 Unnecessary code contributes to larger contract size, which can increase deployment and transaction costs.

## Proof of Concept
In `openCdpOperation` is operation type is set to `openCdpForOperation`
https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router%2Fsrc%2FLeverageZapRouterBase.sol#L165-L178
```solidity
    function _openCdpOperation(
        bytes32 _cdpId,
        OpenCdpForOperation memory _cdp,
        uint256 _flAmount,
        TradeData calldata _tradeData
    ) internal {
        LeverageMacroOperation memory op;

        op.tokenToTransferIn = address(stETH);
        // collateral already transferred in by the caller
        op.amountToTransferIn = 0;
        op.operationType = OperationType.OpenCdpForOperation;
        op.OperationData = abi.encode(_cdp);
        op.swapsAfter = _getSwapOperations(address(ebtcToken), address(stETH), _tradeData);
```


### Defined but Unused Function
https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol%2Fpackages%2Fcontracts%2Fcontracts%2FLeverageMacroBase.sol#L510
```solidity
function _openCdpCallback(bytes memory data) internal virtual {
    OpenCdpOperation memory flData = abi.decode(data, (OpenCdpOperation));

    /**
     * Open CDP and Emit event
     */
    bytes32 _cdpId = borrowerOperations.openCdp(
        flData.eBTCToMint,
        flData._upperHint,
        flData._lowerHint,
        flData.stETHToDeposit
    );
}
```

### Utilized Function
https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol%2Fpackages%2Fcontracts%2Fcontracts%2FLeverageMacroBase.sol#L524
```solidity
function _openCdpForCallback(bytes memory data) internal virtual {
    OpenCdpForOperation memory flData = abi.decode(data, (OpenCdpForOperation));

    /**
     * Open CDP and Emit event
     */
    bytes32 _cdpId = borrowerOperations.openCdpFor(
        flData.eBTCToMint,
        flData._upperHint,
        flData._lowerHint,
        flData.stETHToDeposit,
        flData.borrower
    );
}
```

### Overridden Function with Fee Logic

https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router%2Fsrc%2FLeverageZapRouterBase.sol#L276
```solidity
function _openCdpForCallback(bytes memory data) internal override {
    if (zapFeeBPS > 0) {
        OpenCdpForOperation memory flData = abi.decode(data, (OpenCdpForOperation));

        bytes32 _cdpId = borrowerOperations.openCdpFor(
            flData.eBTCToMint,
            flData._upperHint,
            flData._lowerHint,
            flData.stETHToDeposit,
            flData.borrower
        );

        IERC20(address(ebtcToken)).safeTransfer(zapFeeReceiver, flData.eBTCToMint * zapFeeBPS / BPS);
    } else {
        super._openCdpForCallback(data);
    }
}
```



## Tools Used
Manual Review 

## Recommended Mitigation Steps
 If the `openCdpCallback` function is not needed, it should be removed to streamline the codebase.




2- Incorrect Event Emission 
https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router%2Fsrc%2FZapRouterBase.sol#L81-L89
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
            stEth.transfer(msg.sender, _stEthVal);
        }
    }
```

The events emitted are based on two conditions whether the user wants to receive `stETH` or wStETH` which is indicated by the user by setting the boolean when adjusting or closing CDP 
However the event emission doesn't clearly define this.
https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router%2Fsrc%2FEbtcZapRouter.sol#L468-L472
```solidity
    function _closeCdpWithPermit(
        bytes32 _cdpId,
        bool _useWstETH,
        PositionManagerPermit calldata _positionManagerPermit
    ) internal {
```


The first  emission is for when the user sets the bool to true meaning they want to receive `WstETH` but the emission shows false indicating otherwise 
```solidity
emit ZapOperationEthVariant(
                _cdpId,
                _operationType,
                false,
                address(wstEth),
                _wstETHVal,
                _stEthVal,
                msg.sender
            );
```

##FIX 
Simply change the` false` to` true`

```solidity
emit ZapOperationEthVariant(
                _cdpId,
                _operationType,
                true,
                address(wstEth),
                _wstETHVal,
                _stEthVal,
                msg.sender
            );
```

3- 
Lack of Zero-Value Check When Opening CDP


 No check for zero values when converting assets (e.g., ETH, WETH) to stETH during the process of opening a Collateralized Debt Position (CDP). 
TThis can result in unnecessary function calls and wasted gas fees if a user unintentionally passes a zero value. Ensuring that zero-value inputs are checked and handled appropriately can prevent such inefficiencies and improve the overall robustness of the contract.



## Proof of Concept
Here are the relevant code snippets where zero-value checks are missing:

### : Conversion of ETH to stETH
```solidity
function _convertRawEthToStETH(uint256 _initialETH) internal returns (uint256) {
    require(msg.value == _initialETH, "EbtcZapRouter: Incorrect ETH amount");
    return _depositRawEthIntoLido(_initialETH);
}

function _depositRawEthIntoLido(uint256 _initialETH) internal returns (uint256) {
    uint256 _balBefore = stEth.balanceOf(address(this));
    payable(address(stEth)).call{value: _initialETH}("");
    uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;
    return _deposit;
}
```

### : Conversion of WETH to stETH
```solidity
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

### : Transfer of stETH from caller
```solidity
function _transferInitialStETHFromCaller(uint256 _initialStETH) internal returns (uint256) {
    uint256 _balBefore = stEth.balanceOf(address(this));
    stEth.transferFrom(msg.sender, address(this), _initialStETH);
    uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;
    return _deposit;
}
```

## Tools Used
Manual code review and static analysis.

## Recommended Mitigation Steps
Introduce checks to revert the transaction if zero values are provided for the conversions or transfers. 

#### Conversion of ETH to stETH
```solidity
function _convertRawEthToStETH(uint256 _initialETH) internal returns (uint256) {
    require(msg.value == _initialETH, "EbtcZapRouter: Incorrect ETH amount");
    require(_initialETH > 0, "EbtcZapRouter: ETH amount must be greater than zero");
    return _depositRawEthIntoLido(_initialETH);
}
```

#### Conversion of WETH to stETH
```solidity
function _convertWrappedEthToStETH(uint256 _initialWETH) internal returns (uint256) {
    require(_initialWETH > 0, "EbtcZapRouter: WETH amount must be greater than zero");
    
    uint256 _wETHBalBefore = wrappedEth.balanceOf(address(this));
    wrappedEth.transferFrom(msg.sender, address(this), _initialWETH);
    uint256 _wETHReiceived = wrappedEth.balanceOf(address(this)) - _wETHBalBefore;

    uint256 _rawETHBalBefore = address(this).balance;
    IWrappedETH(address(wrappedEth)).withdraw(_wETHReiceived);
    uint256 _rawETHConverted = address(this).balance - _rawETHBalBefore;
    return _depositRawEthIntoLido(_rawETHConverted);
}
```

#### Transfer of stETH from caller
```solidity
function _transferInitialStETHFromCaller(uint256 _initialStETH) internal returns (uint256) {
    require(_initialStETH > 0, "EbtcZapRouter: stETH amount must be greater than zero");

    uint256 _balBefore = stEth.balanceOf(address(this));
    stEth.transferFrom(msg.sender, address(this), _initialStETH);
    uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;
    return _deposit;
}
```