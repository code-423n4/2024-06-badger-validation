# `[L-01]` Adjusting a cdp with a margin amount requires twice the minimum change in stETH collateral

## Summary

While adjusting a cdp with a margin amount, the minimum adjustment checks are applied to both the stETH balance change for the operation as well as the margin balance. This implies that a user needs to supply twice the minimum change amount to adjust their cdp.

## Vulnerability Details

The `EbtcLeverageZapRouter::_adjustCdp()` function checks if the stETH balance change and the margin balance are either 0 or greater than or equal to the `MIN_CHANGE` parameter set during deployment. If a user wishes to increase their leverage with a margin amount, they will have to supply an stETH balance and a margin balance of atleast `MIN_CHANGE` amount (supplying margin balance is important to maintain a healthy collateral to debt ratio and avoid being liquidated, especially in a leveraged position). On the other hand, if the user wishes to unwind their leveraged position with a margin amount, they will have to withdraw collateral of size twice the `MIN_CHANGE` amount.

Consider the lines pointed to in the following code segment from `EbtcLeverageZapRouter::_adjustcdp()` function.

```javascript
    function _adjustCdp(
        bytes32 _cdpId,
        AdjustCdpParams memory _params,
        bytes calldata _positionManagerPermit,
        TradeData calldata _tradeData,
        uint256 _zapStEthBalanceBefore
    ) internal nonReentrant {
        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for adjust!");
        _requireZeroOrMinAdjustment(_params.debtChange);
@>      _requireZeroOrMinAdjustment(_params.stEthBalanceChange);
@>      _requireZeroOrMinAdjustment(_params.stEthMarginBalance);

        ...

@>      uint256 marginDecrease = _params.isStEthBalanceIncrease ? 0 : _params.stEthBalanceChange;
        if (!_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {
@>          marginDecrease += _params.stEthMarginBalance;
        }

@>      uint256 marginIncrease = _params.isStEthBalanceIncrease ? _params.stEthBalanceChange : 0;
        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {
@>          marginIncrease += _params.stEthMarginBalance;
        }

        ...
    }
```

The stETH margin increase/decrease value with the added stETH balance change is then passed to the `LeverageZapRouterBase::_adjustCdpOperation()` function, as seen below.

```javascript
    function _adjustCdp(
        bytes32 _cdpId,
        AdjustCdpParams memory _params,
        bytes calldata _positionManagerPermit,
        TradeData calldata _tradeData,
        uint256 _zapStEthBalanceBefore
    ) internal nonReentrant {
        ...

        _adjustCdpOperation({
            _cdpId: _cdpId,
            _flType: _params.isDebtIncrease ? FlashLoanType.stETH : FlashLoanType.eBTC,
            _flAmount: _params.flashLoanAmount,
            _cdp: AdjustCdpOperation({
                _cdpId: _cdpId,
                _EBTCChange: _params.debtChange,
                _isDebtIncrease: _params.isDebtIncrease,
                _upperHint: _params.upperHint,
                _lowerHint: _params.lowerHint,
@>              _stEthBalanceIncrease: marginIncrease,
@>              _stEthBalanceDecrease: marginDecrease
            }),
            debt: debt,
            coll: coll,
            _tradeData: _tradeData
        });

        ...
    }
```

## Impact

While adjusting their cdps with a margin, users will need to do so with an stETH amount which is twice the `MIN_CHANGE` amount.

## Recommended Mitigation

In `EbtcLeverageZapRouter::_adjustCdp()` function, check if the net stETH change for the operation is greater than or equal to the `MIN_CHANGE` amount after including the margin calculation.

Make the following changes,

```diff
    function _adjustCdp(
        bytes32 _cdpId,
        AdjustCdpParams memory _params,
        bytes calldata _positionManagerPermit,
        TradeData calldata _tradeData,
        uint256 _zapStEthBalanceBefore
    ) internal nonReentrant {
        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for adjust!");
        _requireZeroOrMinAdjustment(_params.debtChange);
-       _requireZeroOrMinAdjustment(_params.stEthBalanceChange);
-       _requireZeroOrMinAdjustment(_params.stEthMarginBalance);

        // get debt and coll amounts for post checks
        (uint256 debt, uint256 coll) = ICdpManager(address(cdpManager)).getSyncedDebtAndCollShares(_cdpId);

        if (_positionManagerPermit.length > 0) {
            PositionManagerPermit memory approval = abi.decode(_positionManagerPermit, (PositionManagerPermit));
            _permitPositionManagerApproval(borrowerOperations, approval);
        }

        uint256 marginDecrease = _params.isStEthBalanceIncrease ? 0 : _params.stEthBalanceChange;
        if (!_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {
            marginDecrease += _params.stEthMarginBalance;
        }

        uint256 marginIncrease = _params.isStEthBalanceIncrease ? _params.stEthBalanceChange : 0;
        if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {
            marginIncrease += _params.stEthMarginBalance;
        }

+       _requireZeroOrMinAdjustment(marginIncrease);
+       _requireZeroOrMinAdjustment(marginDecrease);
        _requireNonZeroAdjustment(marginIncrease, _params.debtChange, marginDecrease);
        _requireSingularMarginChange(marginIncrease, marginDecrease);

        _adjustCdpOperation({
            _cdpId: _cdpId,
            _flType: _params.isDebtIncrease ? FlashLoanType.stETH : FlashLoanType.eBTC,
            _flAmount: _params.flashLoanAmount,
            _cdp: AdjustCdpOperation({
                _cdpId: _cdpId,
                _EBTCChange: _params.debtChange,
                _isDebtIncrease: _params.isDebtIncrease,
                _upperHint: _params.upperHint,
                _lowerHint: _params.lowerHint,
                _stEthBalanceIncrease: marginIncrease,
                _stEthBalanceDecrease: marginDecrease
            }),
            debt: debt,
            coll: coll,
            _tradeData: _tradeData
        });
        uint256 _zapStEthBalanceDiff = stEth.balanceOf(address(this)) - _zapStEthBalanceBefore;

        if (_positionManagerPermit.length > 0) {
            borrowerOperations.renouncePositionManagerApproval(msg.sender);
        }

        if (_zapStEthBalanceDiff > 0) {
            _transferStEthToCaller(
                _cdpId, EthVariantZapOperationType.AdjustCdp, _params.useWstETHForDecrease, _zapStEthBalanceDiff
            );
        }
    }
}
```


# `[L-02]` Allow the `zapFeeReceiver` address to be changed by the leverage zap router owner

## Summary

In `LeverageZapRouterBase`, the `zapFeeReceiver` is set as an immutable address and cannot be changed. If the fee receiver is set to an incorrect address during deployment, or if the EOA of the fee receiver is compromised/hacked, all zap fees collected by the leverage zap router will be lost.

## Impact

All zap fees collected may be lost/stolen.

## Recommended Mitigation

Allow the leverage zap router owner to set/change the `zapFeeReceiver` address.

Make the following changes in `LeverageZapRouterBase`,

```diff
abstract contract LeverageZapRouterBase is ZapRouterBase, LeverageMacroBase, ReentrancyGuard, IEbtcLeverageZapRouter {
    using SafeERC20 for IERC20;

    uint256 internal constant PRECISION = 1e18;
    uint256 internal constant BPS = 10000;

    address public immutable theOwner;
    address public immutable DEX;
    uint256 public immutable zapFeeBPS;
-   address public immutable zapFeeReceiver;
+   address public zapFeeReceiver;

+   event ZapFeeReceiverChanged(address newZapFeeReceiver);

    ...

+   function setZapFeeReceiver(address _newZapFeeReceiver) external {
+       require(msg.sender == theOwner, "LeverageZapRouterBase: Not owner");
+       zapFeeReceiver = _newZapFeeReceiver;
+       emit ZapFeeReceiverChanged(_newZapFeeReceiver);
+   }
```


# `[L-03]` Use Openzeppelin's `Ownable2Step` library instead of an immutable owner address

## Summary

The owner is set as an immutable address in `LeverageZapRouterBase`. If the owner's EOA is compromised/hacked, all the admin functions in the leverage zap router can be used maliciously. Since the ownership cannot be transferred to a safer address, this will aggravate the issue.

## Impact

There is less flexibility in terms of transferring/managing admin rights. Also, admin related functions may be susceptible to malicious use in case of a hack. Right now, the admin can only sweep the residual funds in the router, however if functionality to change fee receiver, etc is added later on, this can be a more serious issue.

## Recommended Mitigation

Use Openzeppelin's battle-tested `Ownable2Step` library which allows you to transfer the ownership of the leverage zap router in a 2 step method.

Make the following changes,

```diff
+ import { Ownable, Ownable2Step } from "lib/openzeppelin-contracts/contracts/access/Ownable2Step.sol";

...

- abstract contract LeverageZapRouterBase is ZapRouterBase, LeverageMacroBase, ReentrancyGuard, IEbtcLeverageZapRouter {
+ abstract contract LeverageZapRouterBase is
+   ZapRouterBase,
+   LeverageMacroBase,
+   ReentrancyGuard,
+   Ownable2Step,
+   IEbtcLeverageZapRouter {
    using SafeERC20 for IERC20;

    uint256 internal constant PRECISION = 1e18;
    uint256 internal constant BPS = 10000;

-   address public immutable theOwner;
    address public immutable DEX;
    uint256 public immutable zapFeeBPS;
    address public immutable zapFeeReceiver;

    constructor(IEbtcLeverageZapRouter.DeploymentParams memory params)
        ZapRouterBase(params.borrowerOperations, IERC20(params.wstEth), IERC20(params.weth), IStETH(params.stEth))
        LeverageMacroBase(
            params.borrowerOperations,
            params.activePool,
            params.cdpManager,
            params.ebtc,
            params.stEth,
            params.sortedCdps,
            false // Do not sweep
        )
+       Ownable(params.owner)
    {
        if (params.zapFeeBPS > 0) {
            require(params.zapFeeReceiver != address(0));
        }

-       theOwner = params.owner;
        DEX = params.dex;
        zapFeeBPS = params.zapFeeBPS;
        zapFeeReceiver = params.zapFeeReceiver;

        // Infinite Approvals @TODO: do these stay at max for each token?
        ebtcToken.approve(address(borrowerOperations), type(uint256).max);
        stETH.approve(address(borrowerOperations), type(uint256).max);
        stETH.approve(address(activePool), type(uint256).max);
        stEth.approve(address(wstEth), type(uint256).max);
    }
```


# `[L-04]` Missing 0 amount check in `EbtcLeverageZapRouter::_closeCdp()` function

## Summary

When the residual stETH is sweeped back to the user in `EbtcLeverageZapRouter::_closeCdp()` function, there is no check to see if the amount is greater than 0. If the amount is 0, stETH transfer will succeed, however, if the user has set the `_useWstETH` variable to true, the transfer will fail.

## Impact

User's close cdp operation will fail when the amount to sweep in wstETH is 0, even though the cdp was successfully closed. This leads to wastage of gas.

## Recommended Mitigation

Apply a 0 amount check before sweeping the stETH/wstETH amount back to the user.

```diff
    function _closeCdp(
        bytes32 _cdpId,
        bytes calldata _positionManagerPermit,
        bool _useWstETH,
        TradeData calldata _tradeData
    ) internal nonReentrant {
        
        ...

+       if (_stETHDiff > 0)
        _transferStEthToCaller(_cdpId, EthVariantZapOperationType.CloseCdp, _useWstETH, _stETHDiff);
    }
```


# `[NC-01]` Renouncing position management approval after each cdp operation is not required

## Summary

The `EbtcLeverageZapRouter::_openCdp()`, `EbtcLeverageZapRouter::_closeCdp()`, and `EbtcLeverageZapRouter::_adjustCdp()` functions require approval to manage the user's cdp, and renounce the approval after the operation ends successfully. But since the leverage zap router is always granted a one time approval, renouncing the approval is redundant.

This can be seen in `ZapRouterBase::_permitPositionManagerApproval()` function, which is used by the `EbtcLeverageZapRouter` to obtain position management approval from the user.

```javascript
    function _permitPositionManagerApproval(
        IBorrowerOperations borrowerOperations,
        PositionManagerPermit memory _positionManagerPermit
    ) internal {
        try borrowerOperations.permitPositionManagerApproval(
            msg.sender,
            address(this),
@>          IPositionManagers.PositionManagerApproval.OneTime,
            _positionManagerPermit.deadline,
            _positionManagerPermit.v,
            _positionManagerPermit.r,
            _positionManagerPermit.s
        ) {} catch {
            /// @notice adding try...catch around to mitigate potential permit front-running
            /// see: https://www.trust-security.xyz/post/permission-denied
        }
    }
```

## Impact

Renouncing position management approval each time wastes a bit of gas.

## Recommendation

Remove the redundant renouncement of position management approval.