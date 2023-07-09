- The init function on the Well Contract can use calldata instead of memory to save gas.

Gas savings for Well.addLiquidity

- use calldata on addLiquidty param instead of memory to save some gas

File: /src/Well.sol #L 392 - 399

```diff
function addLiquidity(
-   uint256[] memory tokenAmountsIn,
+   uint256[] calldata tokenAmountsIn,
     uint256 minLpAmountOut,
     address recipient,
          uint256 deadline
 ) external nonReentrant expire(deadline) returns (uint256 lpAmountOut) {
     lpAmountOut = _addLiquidity(tokenAmountsIn, minLpAmountOut, recipient, false);
}
```

| Max    |        |
| ------ | ------ |
| Before | 131592 |
| After  | 131437 |

Gas saving for Well.addLiquidityFeeOnTransfer

- use calldata on addLiquidityFeeOnTransfer param instead of memory to save some gas

File: /src/Well.sol #L 401 - 408

```diff
  function addLiquidityFeeOnTransfer(
-    uint256[] memory tokenAmountsIn,
+   uint256[] calldata tokenAmountsIn,
        uint256 minLpAmountOut,
        address recipient,
        uint256 deadline
    ) external nonReentrant expire(deadline) returns (uint256 lpAmountOut) {
        lpAmountOut = _addLiquidity(tokenAmountsIn, minLpAmountOut, recipient, true);
    }
```

| Max    |       |
| ------ | ----- |
| Before | 80373 |
| After  | 80186 |

Gas saving for Well.getAddLiquidityOut

- use calldata on addLiquidityOut param instead of memory to save some gas

File: /src/Well.sol #L 449 - 456

```diff
 function getAddLiquidityOut(
-  uint256[] memory tokenAmountsIn
+  uint256[] calldata tokenAmountsIn
    )
 external view returns (uint256 lpAmountOut) {
        IERC20[] memory _tokens = tokens();
        uint256[] memory reserves = _getReserves(_tokens.length);
        for (uint256 i; i < _tokens.length; ++i) {
            reserves[i] = reserves[i] + tokenAmountsIn[i];
        }
        lpAmountOut = _calcLpTokenSupply(wellFunction(), reserves) - totalSupply();
    }
```

| Max    |       |
| ------ | ----- |
| Before | 13623 |
| After  | 13312 |
