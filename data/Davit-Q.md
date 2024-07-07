## A. Centralization Risks Across Multiple Contract Components

1. Fee Collection: The zapFeeReceiver address is set immutably in the constructor, with no mechanism to update it.

[Code Reference](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/LeverageZapRouterBase.sol#L26C1-L28C6)

2. Contract Ownership: A single owner address (theOwner) has significant privileges across the system.

Code refernces - [1](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/LeverageZapRouterBase.sol#L24-L49), [2](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/LeverageZapRouterBase.sol#L61-L62)


3. Critical Addresses and Parameters: Several critical addresses and parameters are set in the constructor and cannot be changed after deployment.

Code references - [1](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/LeverageZapRouterBase.sol#L22-L28), [2](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/LeverageZapRouterBase.sol#L50-L52), [3](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L51-L59)

4. Lack of Governance Mechanism: There's no governance or multi-signature mechanism for updating critical parameters or addresses.

[Code reference](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L21-L23)

Risk:
If any of these centralized points (owner address, fee receiver, DEX address, etc.) are compromised or become unavailable, it could lead to:

1. Inability to update critical system parameters
2. Potential fund lock-up or loss if the fee receiver becomes inaccessible
3. System-wide vulnerability if the owner's private key is compromised
4. Impossibility to adapt to changing market conditions or upgrade the system

## B. Lack of Event Emission for Critical Operations

Several critical functions in the EbtcLeverageZapRouter and LeverageZapRouterBase contracts don't emit events. [For example](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L171-L181)


Risk: Lack of events makes it difficult for off-chain systems to track important changes and operations in the contract.

## C. Incomplete Input Validation

Some functions in EbtcLeverageZapRouter lack comprehensive input validation. [Code reference](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L34-L43):

    function openCdpWithEth(
        uint256 _debt,
        bytes32 _upperHint,
        bytes32 _lowerHint,
        uint256 _stEthLoanAmount,
        uint256 _ethMarginBalance,
        uint256 _stEthDepositAmount,
        bytes calldata _positionManagerPermit,
        TradeData calldata _tradeData
    ) external payable returns (bytes32 cdpId) {
        // ... (function body)
    }

Risk: Lack of input validation could lead to unexpected behavior or potential exploits.

## D. Insufficient Error Handling and Validation for External Price Data

While the contracts don't directly integrate with Chainlink oracles, they rely on external price data for critical operations. The handling of this price data lacks robust error checking and fallback mechanisms, which could lead to issues if the price source fails or provides incorrect data.

1. Code References:

In the EbtcLeverageZapRouter contract, the [_openCdp](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L205-L214) function uses external price data without sufficient validation:
    
    function _openCdp(
        uint256 _debt,
        bytes32 _upperHint,
        bytes32 _lowerHint,
        uint256 _stEthLoanAmount,
        uint256 _collVal,
        uint256 _stEthDepositAmount,
        bytes calldata _positionManagerPermit,
        TradeData calldata _tradeData
    ) internal nonReentrant returns (bytes32 cdpId) {

[_openCdpOperation](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/EbtcLeverageZapRouter.sol#L238-L243):

    _openCdpOperation({
        _cdpId: cdpId,
        _cdp: cdp,
        _flAmount: _stEthLoanAmount,
        _tradeData: _tradeData
    });

    // ... (other code)
}

2. In the LeverageZapRouterBase contract, the [_getPostCheckParams](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/LeverageZapRouterBase.sol#L257-L274) function uses a _collValidationBufferBPS parameter, which is likely related to price checks, but there's no validation of this value:

    function _getPostCheckParams(
        bytes32 _cdpId,
        uint256 _debt,
        uint256 _totalCollateral,
        ICdpManagerData.Status _status,
        uint256 _collValidationBuffer
    ) internal view returns (PostCheckParams memory) {
        return
            PostCheckParams({
                expectedDebt: CheckValueAndType({value: _debt, operator: Operator.equal}),
                expectedCollateral: CheckValueAndType({
                    value:  _totalCollateral * _collValidationBuffer / BPS,
                    operator: Operator.gte
                }),
                cdpId: _cdpId,
                expectedStatus: _status
            });
    }

3. The [TradeData](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-zap-router/src/interface/IEbtcLeverageZapRouter.sol#L45-L51) struct in IEbtcLeverageZapRouter.sol includes an expectedMinOut field, which is likely used for slippage protection based on price data, but there's no mechanism to validate or update this value if it becomes stale:
    
    struct TradeData {
        bytes exchangeData;
        uint256 expectedMinOut;
        bool performSwapChecks;
        uint256 approvalAmount;
        uint256 collValidationBufferBPS;
    }


Risk:

If the price source provides incorrect or manipulated data, it could lead to mispriced CDPs, potentially allowing users to borrow more or less than they should be able to.
In case of a price feed failure, the system might use outdated prices, leading to incorrect liquidations or preventing necessary liquidations.
Without proper validation of the _collValidationBuffer, there's a risk of setting it to an extreme value, which could bypass important safety checks.