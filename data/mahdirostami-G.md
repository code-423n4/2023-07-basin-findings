## 1. Avoid extra computation by first checking if.
By first checking input values, avoid extra computation.

Instances 10:
```git
     constructor(bytes16 _maxPercentIncrease, bytes16 _maxPercentDecrease, uint256 _blockTime, bytes16 _alpha) {
-        LOG_MAX_INCREASE = ABDKMathQuad.ONE.add(_maxPercentIncrease).log_2();
         // _maxPercentDecrease <= 100%
         if (_maxPercentDecrease > ABDKMathQuad.ONE) {
            revert InvalidMaxPercentDecreaseArgument(_maxPercentDecrease);
        }          
+        // ALPHA <= 1
+        if (_alpha > ABDKMathQuad.ONE) { 
+            revert InvalidAArgument(_alpha);}
+
+        LOG_MAX_INCREASE = ABDKMathQuad.ONE.add(_maxPercentIncrease).log_2();
```
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/pumps/MultiFlowPump.sol#L57C3-L57C3](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/pumps/MultiFlowPump.sol#L57C3-L57C3)
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/pumps/MultiFlowPump.sol#L64](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/pumps/MultiFlowPump.sol#L64)
```git
     ) internal returns (uint256 amountOut) {
+        (uint256 i, uint256 j) = _getIJ(_tokens, fromToken, toToken); 
         IERC20[] memory _tokens = tokens();
         uint256[] memory reserves = _updatePumps(_tokens.length);
-        (uint256 i, uint256 j) = _getIJ(_tokens, fromToken, toToken);
```
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L224](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L224)
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L248](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L248)
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L274](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L274)
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L314](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L314)
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L366](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L366)
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L386](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L386)
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L504](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L504)
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L525](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L525)

## 2. Avoid extra computation with better implementation
#### 1.don't wait until finishing for loop
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L738-L749](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L738-L749)
```git
+        //@audit gas avoid extra
         for (uint256 k; k < _tokens.length; ++k) {
             if (iToken == _tokens[k]) {
                 i = k;
@@ -743,6 +750,9 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
                 j = k;
                 foundJ = true;
             }
+            else if(foundI && foundJ){
+                return (i, j);
+            }
         }
         if (!foundI) revert InvalidTokens();
```
##### 2. Duplicate check
In MultiFlowPump.sol we check 
""// If a reserve is 0, then the pump cannot be initialized.""
we recheck in init function.
```solidtiy
                // If a reserve is 0, then the pump cannot be initialized.
                if (reserves[i] == 0) return;
```
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/pumps/MultiFlowPump.sol#L85C1-L86](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/pumps/MultiFlowPump.sol#L85C1-L86)
```
            if (reserves[i] == 0) return;
```
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/pumps/MultiFlowPump.sol#L153](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/pumps/MultiFlowPump.sol#L153)
## 3. Cache array length if use it multiple times.
```
         IERC20[] memory _tokens = tokens();
-        uint256[] memory reserves = new uint256[](_tokens.length);
+        uint256 _tokenslength = _tokens.length;
+        uint256[] memory reserves = new uint256[](_tokenslength);
 
         // Use the balances of the pool instead of the stored reserves.
         // If there is a change in token balances relative to the currently
         // stored reserves, the extra tokens can be shifted into `tokenOut`.
-        for (uint256 i; i < _tokens.length; ++i) {
+        for (uint256 i; i < _tokenslength; ++i) {
             reserves[i] = _tokens[i].balanceOf(address(this));
         }
```
Instances 8:
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L358](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L358)
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L381](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L381)
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L420](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L420)
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L451](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L451)
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L467](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L467)
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L578](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L578)
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L592](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L592)
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L605](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L605)



