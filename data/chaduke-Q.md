QA1. L492 has the wrong comparison operator, as a result, even when ```swapChecks[i].tokenToCheck).balanceOf(address(this) = swapChecks[i].expectedMinOut```, it will still fail since the compariosn operator is ```>``` instead of ```>=```. 


[https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L485-L497](https://github.com/code-423n4/2024-06-badger/blob/9173558ee1ac8a78a7ae0a39b97b50ff0dd9e0f8/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol#L485-L497)



The POC is to add the following function to LeverageZaps.t.sol: 

```javascript

    function testOpenCdp1() public{
        uint256 price = priceFeedMock.fetchPrice(); // price of stETH in terms of eBTC

        uint256 amount = 10000 ether;
        uint256 marginAmount = 12.3 ether;
        uint256 debt = 1.34e18;
        uint256 flAmount = (debt * 1e18) / price;

   
        console2.log("ccccccccccccccccccccccccc");
        console2.log("price: ", price);

        seedActivePool();

        console2.log("$$$$$$$$$$$$$$$$$$$$$$$ \n \n");

        vm.deal(user1, amount);
        vm.startPrank(user1);
        collateral.deposit{value: amount}();
        collateral.approve(address(testWstEth), type(uint256).max);
        IWstETH(testWstEth).wrap(marginAmount); 
        IERC20(testWstEth).approve(address(leverageZapRouter), type(uint256).max);
        vm.stopPrank();

        IEbtcZapRouter.PositionManagerPermit memory pmPermit = createPermit(user1);
        vm.startPrank(user1);
          leverageZapRouter.openCdpWithWstEth(
                debt, // Debt amount
                bytes32(0),
                bytes32(0),
                flAmount,
                marginAmount, // Margin amount
                (flAmount + IWstETH(testWstEth).getStETHByWstETH(marginAmount)),
                abi.encode(pmPermit),
                _getOpenCdpTradeData(debt, flAmount)
        );
        vm.stopPrank();
    }
```

Mitigation: change the comparison operator to ```>=```. 