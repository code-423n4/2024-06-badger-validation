# require()/revert() strings longer than 32 bytes cost extra gas

Strings are broken into 32 byte chunks for operations. Revert error strings over 32 bytes therefore consume extra gas than shorter strings.

```bash
/2024-06-badger/ebtc-zap-router/src/EbtcLeverageZapRouter.sol:282:        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for close!");


/2024-06-badger/ebtc-zap-router/src/EbtcLeverageZapRouter.sol:410:        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcLeverageZapRouter: not owner for adjust!");


/2024-06-badger/ebtc-zap-router/src/EbtcZapRouter.sol:473:        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcZapRouter: not owner for close!");


/2024-06-badger/ebtc-zap-router/src/EbtcZapRouter.sol:500:        require(msg.sender == _getOwnerAddress(_cdpId), "EbtcZapRouter: not owner for adjust!");


/2024-06-badger/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol:402:        require(initiator == address(this), "LeverageMacroReference: wrong initiator for flashloan");


/2024-06-badger/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol:282:            require(check.value >= valueToCheck, "!LeverageMacroReference: gte post check");


/2024-06-badger/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol:284:            require(check.value <= valueToCheck, "!LeverageMacroReference: let post check");


/2024-06-badger/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol:286:            require(check.value == valueToCheck, "!LeverageMacroReference: equal post check");

```


Since there are multiple areas where strings bigger than 32 bytes are used, replacing them with shorter strings will save a significant amount of gas.

## References
https://github.com/code-423n4/2022-01-livepeer-findings/issues/115

https://stackoverflow.com/questions/72100565/if-string-in-require-statement-is-over-32-bytes-is-it-saved-in-2-storage-slots

