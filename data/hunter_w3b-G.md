# Gas Optimization

# Summary

| Number | Optimization Details                                                                                              | Context |
| :----: | :---------------------------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | Reduce Storage Reads in `Well.sol::pumps` function by declaring Local Variables for `ONE_WORD` & `PACKED_ADDRESS` |    2    |
| [G-02] | Some functions should be directly returns the result instead of `storing` it in a variable first                  |   19    |
| [G-03] | Before some functions we should `check` some variables for possible gas save                                      |    5    |
| [G-04] | Duplicate `if()` checks should be refactored to a function to save gas                                            |    5    |
| [G-05] | Use MAX_UINT128 constant instead of `type(uint128).max` in the LibBytes.sol::storeUint128                         |    5    |
| [G-06] | Do not calculate constants                                                                                        |    6    |
| [G-07] | Use do while loops instead of for loops                                                                           |   33    |
| [G-08] | Expressions for constant values such as a call to `keccak256()` should use immutable rather than constant         |    1    |
| [G-09] | Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4                             |    2    |
| [G-10] | `array[index]` += amount is cheaper than `array[index]` = `array[index] + amount`                                 |    4    |
| [G-11] | if possible use for loop in Yul this will save significant amount of gas                                          |   33    |
| [G-12] | Use solidity version 0.8.20 to gain some gas boost                                                                |   10    |
| [G-13] | With assembly, `.call (bool success)` transfer can be done gas-optimized                                          |    1    |
| [G-14] | `x += y` costs more gas than `x = x + y` for state variables                                                      |    2    |
| [G-15] | String literals passed to `abi.encode()/abi.encodePacked()` should not be split by commas                         |    1    |
| [G-16] | Remove or replace unused variables                                                                                |    1    |
| [G-17] | Using a positive conditional flow to save a `NOT` opcode                                                          |    2    |

## [G-01] Reduce Storage Reads in `Well.sol::pumps` function by declaring Local Variables for `ONE_WORD` & `PACKED_ADDRESS`

In the Well.sol::pumps function, consider declaring a local variable to store the value of the `ONE_WORD` state variable, which is read twice from storage.

Once during the initialization of the `dataLoc` variable, and again within the loop. This can lead to excessive storage reads, especially if the loop iterates a significant number of times based on the length of `_pumps`. Additionally, the `PACKED_ADDRESS` state variable is also used within the loop, further contributing to storage reads. To improve gas efficiency, declare separate local variables for both `ONE_WORD` and `PACKED_ADDRESS` outside the loop, assign their values to these variables, and then use the local variables within the loop.
This optimization reduces the number of storage reads and gas savings.

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol

```solidity
File: src/Well.sol

94    function pumps() public pure returns (Call[] memory _pumps) {
95        if (numberOfPumps() == 0) return _pumps;
96
97        _pumps = new Call[](numberOfPumps());
98        uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD + wellFunctionDataLength();
99
100        uint256 pumpDataLength;
101        for (uint256 i; i < _pumps.length; i++) {
102            _pumps[i].target = _getArgAddress(dataLoc);
103            dataLoc += PACKED_ADDRESS;
104            pumpDataLength = _getArgUint256(dataLoc);
105            dataLoc += ONE_WORD;
106            _pumps[i].data = _getArgBytes(dataLoc, pumpDataLength);
107            dataLoc += pumpDataLength;
108       }
109    }
```

```diff
diff --git a/Well.org.sol b/Well.sol
index 5f4d57d..764f95e 100644
--- a/Well.org.sol
+++ b/Well.sol
@@ -1,15 +1,16 @@
     function pumps() public pure returns (Call[] memory _pumps) {
+        uint256 ONE_WORD = _one_word
+        uint256 PACKED_ADDRESS = _packed_address
         if (numberOfPumps() == 0) return _pumps;

         _pumps = new Call[](numberOfPumps());
-        uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD + wellFunctionDataLength();
+        uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * _one_word + wellFunctionDataLength();

         uint256 pumpDataLength;
         for (uint256 i; i < _pumps.length; i++) {
             _pumps[i].target = _getArgAddress(dataLoc);
-             dataLoc += PACKED_ADDRESS;
+             dataLoc += _packed_address;
             pumpDataLength = _getArgUint256(dataLoc);
-            dataLoc += ONE_WORD;
+            dataLoc += _one_word;
             _pumps[i].data = _getArgBytes(dataLoc, pumpDataLength);
             dataLoc += pumpDataLength;
         }

```

## [G-02] Some functions should be directly returns the result instead of `storing` it in a variable first

The function directly returns the result of `_swapFrom` instead of storing it in a variable first. This can save gas by avoiding an extra storage operation.

`Well.sol::swapFrom` function should be directly returns the result of `_swapFrom` instead of storing it in a variable first

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L194

```solidity
File: src/Well.sol

186  function swapFrom(
187        IERC20 fromToken,
188        IERC20 toToken,
189        uint256 amountIn,
190        uint256 minAmountOut,
191        address recipient,
192        uint256 deadline
193    ) external nonReentrant expire(deadline) returns (uint256 amountOut) {
194        fromToken.safeTransferFrom(msg.sender, address(this), amountIn);
195        amountOut = _swapFrom(fromToken, toToken, amountIn, minAmountOut, recipient);
196    }
```

```diff
diff --git a/Well.org.sol b/Well.sol
index 1cce6b1..2c75f76 100644
--- a/Well.org.sol
+++ b/Well.sol
@@ -7,5 +7,8 @@
         uint256 deadline
     ) external nonReentrant expire(deadline) returns (uint256 amountOut) {
         fromToken.safeTransferFrom(msg.sender, address(this), amountIn);
-        amountOut = _swapFrom(fromToken, toToken, amountIn, minAmountOut, recipient);
+        return _swapFrom(fromToken, toToken, amountIn, minAmountOut, recipient);
    }

```

`Well.sol::swapFromFeeOnTransfer` function should be directly returns the result of `_swapFrom` instead of storing it in a variable first

```solidity
File: src/Well.sol
    function swapFromFeeOnTransfer(
        IERC20 fromToken,
        IERC20 toToken,
        uint256 amountIn,
        uint256 minAmountOut,
        address recipient,
        uint256 deadline
    ) external nonReentrant expire(deadline) returns (uint256 amountOut) {
        amountIn = _safeTransferFromFeeOnTransfer(fromToken, msg.sender, amountIn);
        amountOut = _swapFrom(fromToken, toToken, amountIn, minAmountOut, recipient);
    }
```

```diff
diff --git a/Well.org.sol b/Well.sol
index 36f0578..dfe55da 100644
--- a/Well.org.sol
+++ b/Well.sol
@@ -7,5 +7,5 @@
         uint256 deadline
     ) external nonReentrant expire(deadline) returns (uint256 amountOut) {
         amountIn = _safeTransferFromFeeOnTransfer(fromToken, msg.sender, amountIn);
-        amountOut = _swapFrom(fromToken, toToken, amountIn, minAmountOut, recipient);
+        return  _swapFrom(fromToken, toToken, amountIn, minAmountOut, recipient);
     }

```

`Well.sol::getSwapOut` function directly should be returns the result of `reserves[j] - _calcReserve(wellFunction(), reserves, j, totalSupply());` instead of storing it in a variable first

```solidity

245    function getSwapOut(IERC20 fromToken, IERC20 toToken, uint256 amountIn) external view returns (uint256 amountOut) {
246        IERC20[] memory _tokens = tokens();
247        uint256[] memory reserves = _getReserves(_tokens.length);
248        (uint256 i, uint256 j) = _getIJ(_tokens, fromToken, toToken);
249
250        reserves[i] += amountIn;
251
252        // underflow is desired; Well Function SHOULD NOT increase reserves of both `i` and `j`
253        amountOut = reserves[j] - _calcReserve(wellFunction(), reserves, j, totalSupply());
254    }
```

```diff
diff --git a/Well.org.sol b/Well.sol
index 19e10ce..2a30042 100644
--- a/Well.org.sol
+++ b/Well.sol
@@ -6,5 +6,5 @@
         reserves[i] += amountIn;

         // underflow is desired; Well Function SHOULD NOT increase reserves of both `i` and `j`
-        amountOut = reserves[j] - _calcReserve(wellFunction(), reserves, j, totalSupply());
+        return reserves[j] - _calcReserve(wellFunction(), reserves, j, totalSupply());
     }

```

```solidity

311    function getSwapIn(IERC20 fromToken, IERC20 toToken, uint256 amountOut) external view returns (uint256 amountIn) {
312        IERC20[] memory _tokens = tokens();
313        uint256[] memory reserves = _getReserves(_tokens.length);
314        (uint256 i, uint256 j) = _getIJ(_tokens, fromToken, toToken);
315
316        reserves[j] -= amountOut;
317
318        amountIn = _calcReserve(wellFunction(), reserves, i, totalSupply()) - reserves[i];
319    }
```

```diff
diff --git a/Well.org.sol b/Well.sol
index 3d04f1c..da7414c 100644
--- a/Well.org.sol
+++ b/Well.sol
@@ -5,5 +5,5 @@

         reserves[j] -= amountOut;

-        amountIn = _calcReserve(wellFunction(), reserves, i, totalSupply()) - reserves[i];
+        return _calcReserve(wellFunction(), reserves, i, totalSupply()) - reserves[i];
     }
```

```solidity
File: src/Well.sol

398        lpAmountOut = _addLiquidity(tokenAmountsIn, minLpAmountOut, recipient, false);

407        lpAmountOut = _addLiquidity(tokenAmountsIn, minLpAmountOut, recipient, true);

455        lpAmountOut = _calcLpTokenSupply(wellFunction(), reserves) - totalSupply();

490        tokenAmountsOut = _calcLPTokenUnderlying(wellFunction(), lpAmountIn, reserves, lpTokenSupply);

526        tokenAmountOut = _getRemoveLiquidityOneTokenOut(lpAmountIn, j, reserves);

543        tokenAmountOut = reserves[j] - newReserveJ;

582        lpAmountIn = totalSupply() - _calcLpTokenSupply(wellFunction(), reserves);

618        reserves = _getReserves(numberOfTokens());

625        reserves = LibBytes.readUint128(RESERVES_STORAGE_SLOT, _numberOfTokens);

685        lpTokenSupply = IWellFunction(_wellFunction.target).calcLpTokenSupply(reserves, _wellFunction.data);

701        reserve = IWellFunction(_wellFunction.target).calcReserve(reserves, j, lpTokenSupply, _wellFunction.data);


719        tokenAmounts = IWellFunction(_wellFunction.target).calcLPTokenUnderlying(

781        amountTransferred = token.balanceOf(address(this)) - balanceBefore;

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol

```solidity
File: libraries/LibWellConstructor.sol

52        immutableData = abi.encodePacked(

81        initFunctionCall = abi.encodeWithSignature("init(string,string)", name, symbol);

```

## [G-03] Before some functions we should check some variables for possible gas save

in the `Well.sol::swapFrom` function we should check for amount being 0 so the function doesn't run when its not gonna do anything.
it's important to note that the `safeTransferFrom` function being called on fromToken may have its own checks of zero amount, but it's not a good idea to call the function that not gonna do anything.

```solidity
File: src/Well.sol

186  function swapFrom(
187        IERC20 fromToken,
188        IERC20 toToken,
189        uint256 amountIn,
190        uint256 minAmountOut,
191        address recipient,
192        uint256 deadline
193    ) external nonReentrant expire(deadline) returns (uint256 amountOut) {
194        fromToken.safeTransferFrom(msg.sender, address(this), amountIn);
195        amountOut = _swapFrom(fromToken, toToken, amountIn, minAmountOut, recipient);
196    }
```

```diff
diff --git a/Well.org.sol b/Well.sol
index 3d1fd95..e51d93b 100644
--- a/Well.org.sol
+++ b/Well.sol
@@ -6,6 +6,6 @@
         address recipient,
         uint256 deadline
     ) external nonReentrant expire(deadline) returns (uint256 amountOut) {
-        fromToken.safeTransferFrom(msg.sender, address(this), amountIn);
+        amountIn != 0 ? fromToken.safeTransferFrom(msg.sender, address(this), amountIn) : "zero amount";
         amountOut = _swapFrom(fromToken, toToken, amountIn, minAmountOut, recipient);
   }

```

`Well.sol::swapFromFeeOnTransfer`

```diff
File: src/Well.sol

203   function swapFromFeeOnTransfer(
204        IERC20 fromToken,
205        IERC20 toToken,
206        uint256 amountIn,
207        uint256 minAmountOut,
208        address recipient,
209        uint256 deadline
210    ) external nonReentrant expire(deadline) returns (uint256 amountOut) {
+          require(amountIn != 0, "zero amount");
211        amountIn = _safeTransferFromFeeOnTransfer(fromToken, msg.sender, amountIn);
212        amountOut = _swapFrom(fromToken, toToken, amountIn, minAmountOut, recipient);
213    }
```

```diff
File: src/Well.sol

245    function getSwapOut(IERC20 fromToken, IERC20 toToken, uint256 amountIn) external view returns (uint256 amountOut) {

+          require(amountIn != 0, "zero amount");
246        IERC20[] memory _tokens = tokens();
247        uint256[] memory reserves = _getReserves(_tokens.length);
248        (uint256 i, uint256 j) = _getIJ(_tokens, fromToken, toToken);
249
250        reserves[i] += amountIn;
251
252        // underflow is desired; Well Function SHOULD NOT increase reserves of both `i` and `j`
253        amountOut = reserves[j] - _calcReserve(wellFunction(), reserves, j, totalSupply());
254    }

```

```diff

264    function swapTo(
265        IERC20 fromToken,
266        IERC20 toToken,
267        uint256 maxAmountIn,
268        uint256 amountOut,
269        address recipient,
270        uint256 deadline
271    ) external nonReentrant expire(deadline) returns (uint256 amountIn) {
+          require(amountOut != 0, "zero amount");
272        IERC20[] memory _tokens = tokens();
273        uint256[] memory reserves = _updatePumps(_tokens.length);
274        (uint256 i, uint256 j) = _getIJ(_tokens, fromToken, toToken);
275
276        reserves[j] -= amountOut;
277        uint256 reserveIBefore = reserves[i];
278        reserves[i] = _calcReserve(wellFunction(), reserves, i, totalSupply());
```

```diff

311    function getSwapIn(IERC20 fromToken, IERC20 toToken, uint256 amountOut) external view returns (uint256 amountIn) {
+          require(amountOut != 0, "zero amount");
312        IERC20[] memory _tokens = tokens();
313        uint256[] memory reserves = _getReserves(_tokens.length);
314        (uint256 i, uint256 j) = _getIJ(_tokens, fromToken, toToken);
315
316        reserves[j] -= amountOut;
317
318        amountIn = _calcReserve(wellFunction(), reserves, i, totalSupply()) - reserves[i];
319    }

```

## [G-04] Duplicated `if()` checks should be refactored to a function to save gas

Extracting repetitive if() checks into a separate function, which can help reduce gas.

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol

```solidity
File: pumps/MultiFlowPump.sol

174        if (numberOfReserves == 0) {

225        if (numberOfReserves == 0) {

243        if (numberOfReserves == 0) {

270        if (numberOfReserves == 0) {

289        if (numberOfReserves == 0) {

```

```diff

diff --git a/MultiFlowPump.org.sol b/MultiFlowPump.sol
index 5b682e6..7eeb036 100644
--- a/MultiFlowPump.org.sol
+++ b/MultiFlowPump.sol
@@ -1,11 +1,14 @@
     function readLastReserves(address well) public view returns (uint256[] memory reserves) {
         bytes32 slot = _getSlotForAddress(well);
         (uint8 numberOfReserves,, bytes16[] memory bytesReserves) = slot.readLastReserves();
-        if (numberOfReserves == 0) {
-            revert NotInitialized();
-        }
+        checkIfInitialized(numberOfReserves);
         reserves = new uint256[](numberOfReserves);
         for (uint256 i; i < numberOfReserves; ++i) {
             reserves[i] = bytesReserves[i].pow_2ToUInt();
         }
    }

+    function checkIfInitialized(uint8 numberOfReserves) private pure {
+        if (numberOfReserves == 0) {
+           revert NotInitialized();
     }
```

```diff

222    function readLastInstantaneousReserves(address well) public view returns (uint256[] memory reserves) {
223        bytes32 slot = _getSlotForAddress(well);
224        uint8 numberOfReserves = slot.readNumberOfReserves();
-225        if (numberOfReserves == 0) {
-226            revert NotInitialized();
-227        }
+          checkIfInitialized(numberOfReserves);
228        uint256 offset = _getSlotsOffset(numberOfReserves);
229        assembly {
230            slot := add(slot, offset)
231        }
232        bytes16[] memory byteReserves = slot.readBytes16(numberOfReserves);
233        reserves = new uint256[](numberOfReserves);
234        for (uint256 i; i < numberOfReserves; ++i) {
235            reserves[i] = byteReserves[i].pow_2ToUInt();
236        }
237    }



239    function readInstantaneousReserves(address well, bytes memory) public view returns (uint256[] memory emaReserves) {
240        bytes32 slot = _getSlotForAddress(well);
241        uint256[] memory reserves = IWell(well).getReserves();
242        (uint8 numberOfReserves, uint40 lastTimestamp, bytes16[] memory lastReserves) = slot.readLastReserves();
-243        if (numberOfReserves == 0) {
-244            revert NotInitialized();
-245        }
+          checkIfInitialized(numberOfReserves);
246        uint256 offset = _getSlotsOffset(numberOfReserves);
247        assembly {
248            slot := add(slot, offset)
249        }




269        uint8 numberOfReserves = slot.readNumberOfReserves();
-270        if (numberOfReserves == 0) {
-271            revert NotInitialized();
-272        }
+          checkIfInitialized(numberOfReserves);
273        uint256 offset = _getSlotsOffset(numberOfReserves) << 1;





288        (uint8 numberOfReserves, uint40 lastTimestamp, bytes16[] memory lastReserves) = slot.readLastReserves();
-289        if (numberOfReserves == 0) {
-290            revert NotInitialized();
-291        }
+          checkIfInitialized(numberOfReserves);
292        uint256 offset = _getSlotsOffset(numberOfReserves) << 1;
```

## [G-05] Use MAX_UINT128 constant instead of `type(uint128).max` in the LibBytes.sol::storeUint128

type(uint128).max it uses more gas in the distribution process and also for each transaction than constant usage.

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol

```solidity
File: libraries/LibBytes.sol

40            require(reserves[0] <= type(uint128).max, "ByteStorage: too large");

41            require(reserves[1] <= type(uint128).max, "ByteStorage: too large");

49                require(reserves[2 * i] <= type(uint128).max, "ByteStorage: too large");

50                require(reserves[2 * i + 1] <= type(uint128).max, "ByteStorage: too large");

63                require(reserves[reserves.length - 1] <= type(uint128).max, "ByteStorage: too large");


```

```diff

diff --git a/LibBytes.org.sol b/LibBytes.sol
index 504e44c..3122131 100644
--- a/LibBytes.org.sol
+++ b/LibBytes.sol
@@ -1,8 +1,11 @@
+    uint128 constant MAX_UINT128 = type(uint128).max;

     function storeUint128(bytes32 slot, uint256[] memory reserves) internal {
         // Shortcut: two reserves can be packed into one slot without a loop
+        uint128 max_uint128 = MAX_UINT128
         if (reserves.length == 2) {
-            require(reserves[0] <= type(uint128).max, "ByteStorage: too large");
-            require(reserves[1] <= type(uint128).max, "ByteStorage: too large");
+            require(reserves[0] <= max_uint128, "ByteStorage: too large");
+            require(reserves[1] <= max_uint128, "ByteStorage: too large");
             assembly {
                 sstore(slot, add(mload(add(reserves, 32)), shl(128, mload(add(reserves, 64)))))
             }
             uint256 maxI = reserves.length / 2; // number of fully-packed slots
             uint256 iByte; // byte offset of the current reserve
             for (uint256 i; i < maxI; ++i) {
-                require(reserves[2 * i] <= type(uint128).max, "ByteStorage: too large");
-                require(reserves[2 * i + 1] <= type(uint128).max, "ByteStorage: too large");
+                require(reserves[2 * i] <= max_uint128, "ByteStorage: too large");
+                require(reserves[2 * i + 1] <= max_uint128, "ByteStorage: too large");
                 iByte = i * 64;
```

## [G-06] Do not calculate constants

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L29

```solidity
File: src/Well.sol

29    bytes32 constant RESERVES_STORAGE_SLOT = bytes32(uint256(keccak256("reserves.storage.slot")) - 1);

78    uint256 constant LOC_TOKENS_COUNT = LOC_AQUIFER_ADDR + PACKED_ADDRESS;

79    uint256 constant LOC_WELL_FUNCTION_ADDR = LOC_TOKENS_COUNT + ONE_WORD;

80    uint256 constant LOC_WELL_FUNCTION_DATA_LENGTH = LOC_WELL_FUNCTION_ADDR + PACKED_ADDRESS;

81    uint256 constant LOC_PUMPS_COUNT = LOC_WELL_FUNCTION_DATA_LENGTH + ONE_WORD;

82    uint256 constant LOC_VARIABLE = LOC_PUMPS_COUNT + ONE_WORD;

```

## [G-07] Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L116

```solidity
File: pumps/MultiFlowPump.sol

84            for (uint256 i; i < numberOfReserves; ++i) {


116        for (uint256 i; i < numberOfReserves; ++i) {


152        for (uint256 i; i < numberOfReserves; ++i) {


178        for (uint256 i; i < numberOfReserves; ++i) {


234        for (uint256 i; i < numberOfReserves; ++i) {


255        for (uint256 i; i < numberOfReserves; ++i) {


301        for (uint256 i; i < cumulativeReserves.length; ++i) {


322        for (uint256 i; i < byteCumulativeReserves.length; ++i) {

```

```diff
diff --git a/MultiFlowPump.org.sol b/MultiFlowPump.sol
index a3730ec..4ed2dc7 100644
--- a/MultiFlowPump.org.sol
+++ b/MultiFlowPump.sol
@@ -1,4 +1,5 @@

-            for (uint256 i; i < numberOfReserves; ++i) {
+        uint256 i;
+        do{
                // If a reserve is 0, then the pump cannot be initialized.
                if (reserves[i] == 0) return;
-            }
+         } while(i < numberOfReserves);


-        for (uint256 i; i < numberOfReserves; ++i) {
+        uint256 i;
+        do{
             // Use a minimum of 1 for reserve. Geometric means will be set to 0 if a reserve is 0.
             pumpState.lastReserves[i] = _capReserve(
                 pumpState.lastReserves[i], (reserves[i] > 0 ? reserves[i] : 1).fromUIntToLog2(), blocksPassed
@@ -7,4 +8,4 @@
                 pumpState.lastReserves[i].mul((ABDKMathQuad.ONE.sub(alphaN))).add(pumpState.emaReserves[i].mul(alphaN));
             pumpState.cumulativeReserves[i] =
                 pumpState.cumulativeReserves[i].add(pumpState.lastReserves[i].mul(deltaTimestampBytes));
-        }
+         } while(i < numberOfReserves);




-        for (uint256 i; i < numberOfReserves; ++i) {
+        uint256 i;
+        do{
             if (reserves[i] == 0) return;
             byteReserves[i] = reserves[i].fromUIntToLog2();
-           }
+        } while(i < numberOfReserves);





-        for (uint256 i; i < numberOfReserves; ++i) {
+        uint256 i;
+        do{
            reserves[i] = bytesReserves[i].pow_2ToUInt();
        }
+        } while(i < numberOfReserves);




-        for (uint256 i; i < numberOfReserves; ++i) {
+        uint256 i;
+        do{
            reserves[i] = bytesReserves[i].pow_2ToUInt();
-        }
+        } while(i < numberOfReserves);




-        for (uint256 i; i < numberOfReserves; ++i) {
+        uint256 i;
+        do{
            lastReserves[i] = _capReserve(lastReserves[i], reserves[i].fromUIntToLog2(), blocksPassed);
            emaReserves[i] =
                lastReserves[i].mul((ABDKMathQuad.ONE.sub(alphaN))).add(lastEmaReserves[i].mul(alphaN)).pow_2ToUInt();
-        }
+        } while(i < numberOfReserves);





-        for (uint256 i; i < cumulativeReserves.length; ++i) {
+        uint256 i;
+        do{
            lastReserves[i] = _capReserve(lastReserves[i], reserves[i].fromUIntToLog2(), blocksPassed);
            cumulativeReserves[i] = cumulativeReserves[i].add(lastReserves[i].mul(deltaTimestampBytes));
-        }
+        } while(i < numberOfReserves);




-        for (uint256 i; i < byteCumulativeReserves.length; ++i) {
+        uint256 i;
+        do{
            // Currently, there is no support for overflow.
            twaReserves[i] =
                (byteCumulativeReserves[i].sub(byteStartCumulativeReserves[i])).div(deltaTimestamp).pow_2ToUInt();
-        }
+        } while(i < numberOfReserves);
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol

```solidity
File:  src/Well.sol

36        for (uint256 i; i < _tokens.length - 1; ++i) {
37            for (uint256 j = i + 1; j < _tokens.length; ++j) {


101        for (uint256 i; i < _pumps.length; i++) {


363        for (uint256 i; i < _tokens.length; ++i) {


382        for (uint256 i; i < _tokens.length; ++i) {


423            for (uint256 i; i < _tokens.length; ++i) {


429            for (uint256 i; i < _tokens.length; ++i) {

452        for (uint256 i; i < _tokens.length; ++i) {


473        for (uint256 i; i < _tokens.length; ++i) {


557        for (uint256 i; i < _tokens.length; ++i) {


549        for (uint256 i; i < _tokens.length; ++i) {


593        for (uint256 i; i < _tokens.length; ++i) {

607        for (uint256 i; i < _tokens.length; ++i) {


633        for (uint256 i; i < reserves.length; ++i) {


662            for (uint256 i; i < _pumps.length; ++i) {


738        for (uint256 k; k < _tokens.length; ++k) {


760        for (j; j < _tokens.length; ++j) {

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L73

```solidity
File: LibWellConstructor.sol

73        for (uint256 i = 1; i < _tokens.length; ++i) {


43        for (uint256 i; i < _pumps.length; ++i) {

```

```diff
diff --git a/LibWellConstructor.org.sol b/LibWellConstructor.sol
index 411715f..bfa4e2d 100644
--- a/LibWellConstructor.org.sol
+++ b/LibWellConstructor.sol
@@ -1,4 +1,6 @@


-         for (uint256 i = 1; i < _tokens.length; ++i) {
+        uint256 i = 1;
+        do{
             name = string.concat(name, ":", LibContractInfo.getSymbol(address(_tokens[i])));
             symbol = string.concat(symbol, LibContractInfo.getSymbol(address(_tokens[i])));
-        }
+        } while(i < _tokens.length)




-        for (uint256 i; i < _pumps.length; ++i) {
+        uint256 i = 1;
+        do{
            packedPumps = abi.encodePacked(
                packedPumps,            // previously packed pumps
                _pumps[i].target,       // pump address
                _pumps[i].data.length,  // pump data length
                _pumps[i].data          // pump data (bytes)
            );
        }
+        } while(i < _pumps.length)
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol#L41

```solidity
File: LibLastReserveBytes.sol

41            for (uint256 i = 1; i < maxI; ++i) {

93            for (uint256 i = 3; i <= n; ++i) {

```

```diff
diff --git a/LibLastReserveBytes.org.sol b/LibLastReserveBytes.sol
index c25be3c..e8cc5ba 100644
--- a/LibLastReserveBytes.org.sol
+++ b/LibLastReserveBytes.sol
@@ -1,9 +1,12 @@

             uint256 iByte; // byte offset of the current reserve
-             for (uint256 i = 1; i < maxI; ++i) {
+            uint256 i = 1;
+            do{
                 iByte = i * 64;
                 assembly {
                     sstore(
                         add(slot, mul(i, 32)),
                         add(mload(add(reserves, add(iByte, 32))), shr(128, mload(add(reserves, add(iByte, 64)))))
                     )
-                }
+            }while(i < maxI);
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes16.sol

```solidity
File: libraries/LibBytes16.sol

69        for (uint256 i = 1; i <= n; ++i) {

28            for (uint256 i; i < maxI; ++i) {

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L48

```solidity
File: libraries/LibBytes.sol

48            for (uint256 i; i < maxI; ++i) {

92        for (uint256 i = 1; i <= n; ++i) {

```

```diff
diff --git a/org.sol b/not.sol
index 6f1743a..cd55f33 100644
--- a/org.sol
+++ b/not.sol
@@ -1,4 +1,6 @@
-       for (uint256 i = 1; i <= n; ++i) {
+      uint256 i = 1;
+      do {
             // `iByte` is the byte position for the current slot:
             // i        1 2 3 4 5 6
             // iByte    0 0 1 1 2 2
@@ -17,4 +19,5 @@
                     mstore(add(reserves, mul(i, 32)), shr(128, sload(add(slot, iByte))))
                 }
             }
-        }
+    } while(i <= n);

```

## [G-08] Expressions for constant values such as a call to `keccak256()` should use immutable rather than constant

It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once.

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L29

```solidity
File: src/Well.sol

29    bytes32 constant RESERVES_STORAGE_SLOT = bytes32(uint256(keccak256("reserves.storage.slot")) - 1);

```

## [G-09] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

he bytes.concat() function is more optimized in terms of gas usage compared to abi.encodePacked(). It avoids unnecessary copying and padding, resulting in potentially lower gas costs for concatenation operations.

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol

```solidity
File: libraries/LibWellConstructor.sol

44            packedPumps = abi.encodePacked(
45                packedPumps,            // previously packed pumps
46                _pumps[i].target,       // pump address
47                _pumps[i].data.length,  // pump data length
48                _pumps[i].data          // pump data (bytes)
49            );



52        immutableData = abi.encodePacked(
53            _aquifer,                   // aquifer address
54            _tokens.length,             // number of tokens
55            _wellFunction.target,       // well function address
56            _wellFunction.data.length,  // well function data length
57            _pumps.length,              // number of pumps
58            _tokens,                    // tokens array
59            _wellFunction.data,         // well function data (bytes)
60            packedPumps                 // packed pumps (bytes)
61        );
```

## [G-10] `array[index]` += amount is cheaper than `array[index]` = `array[index] + amount`

When you use `array[index]` += amount, Solidity can sometimes perform an in-place modification of the array element. This means that the value at `array[index]` is updated directly without needing to load the existing value into a temporary variable. This optimization can save gas because it avoids the additional step of reading the value from memory before performing the addition.

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L226

```solidity
File: src/Well.sol

226        reserves[i] += amountIn;


250        reserves[i] += amountIn;


276        reserves[j] -= amountOut;


316        reserves[j] -= amountOut;

```

## [G-11] if possible use for loop in Yul this will save significant amount of gas

https://twitter.com/0xkato/status/1669072272534175746

For Example:

```solidity

    uint256 OddNumbers;

    for (uint  i = x; i < y +1 ; ++i) {
	    if (i % 2 == 1){
		    ++OddNumbers;
		    }
    }


    uint256 OddNumbers;

    assembly {

        for { let i := x } lt( i, add(y, 1)  ) { i := add(i,1) }
        {
            // checks if i % 2 == 1
            if  eq( mod( i, 2 ), 1 ) {
                OddNumbers := add(OddNumbers, 1)
            }
        }
    }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L116

```solidity
File: pumps/MultiFlowPump.sol

84            for (uint256 i; i < numberOfReserves; ++i) {


116        for (uint256 i; i < numberOfReserves; ++i) {


152        for (uint256 i; i < numberOfReserves; ++i) {


178        for (uint256 i; i < numberOfReserves; ++i) {


234        for (uint256 i; i < numberOfReserves; ++i) {


255        for (uint256 i; i < numberOfReserves; ++i) {


301        for (uint256 i; i < cumulativeReserves.length; ++i) {


322        for (uint256 i; i < byteCumulativeReserves.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol

```solidity
File:  src/Well.sol

36        for (uint256 i; i < _tokens.length - 1; ++i) {
37            for (uint256 j = i + 1; j < _tokens.length; ++j) {


101        for (uint256 i; i < _pumps.length; i++) {


363        for (uint256 i; i < _tokens.length; ++i) {


382        for (uint256 i; i < _tokens.length; ++i) {


423            for (uint256 i; i < _tokens.length; ++i) {


429            for (uint256 i; i < _tokens.length; ++i) {

452        for (uint256 i; i < _tokens.length; ++i) {


473        for (uint256 i; i < _tokens.length; ++i) {


557        for (uint256 i; i < _tokens.length; ++i) {


549        for (uint256 i; i < _tokens.length; ++i) {


593        for (uint256 i; i < _tokens.length; ++i) {

607        for (uint256 i; i < _tokens.length; ++i) {


633        for (uint256 i; i < reserves.length; ++i) {


662            for (uint256 i; i < _pumps.length; ++i) {


738        for (uint256 k; k < _tokens.length; ++k) {


760        for (j; j < _tokens.length; ++j) {

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes16.sol

```solidity
File: libraries/LibBytes16.sol

69        for (uint256 i = 1; i <= n; ++i) {

28            for (uint256 i; i < maxI; ++i) {

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L48

```solidity
File: libraries/LibBytes.sol

48            for (uint256 i; i < maxI; ++i) {

92        for (uint256 i = 1; i <= n; ++i) {

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol#L41

```solidity
File: LibLastReserveBytes.sol

41            for (uint256 i = 1; i < maxI; ++i) {

93            for (uint256 i = 3; i <= n; ++i) {

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol

```solidity
File:  src/Well.sol

36        for (uint256 i; i < _tokens.length - 1; ++i) {
37            for (uint256 j = i + 1; j < _tokens.length; ++j) {


101        for (uint256 i; i < _pumps.length; i++) {


363        for (uint256 i; i < _tokens.length; ++i) {


382        for (uint256 i; i < _tokens.length; ++i) {


423            for (uint256 i; i < _tokens.length; ++i) {


429            for (uint256 i; i < _tokens.length; ++i) {

452        for (uint256 i; i < _tokens.length; ++i) {


473        for (uint256 i; i < _tokens.length; ++i) {


557        for (uint256 i; i < _tokens.length; ++i) {


549        for (uint256 i; i < _tokens.length; ++i) {


593        for (uint256 i; i < _tokens.length; ++i) {

607        for (uint256 i; i < _tokens.length; ++i) {


633        for (uint256 i; i < reserves.length; ++i) {


662            for (uint256 i; i < _pumps.length; ++i) {


738        for (uint256 k; k < _tokens.length; ++k) {


760        for (j; j < _tokens.length; ++j) {

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L73

```solidity
File: LibWellConstructor.sol

73        for (uint256 i = 1; i < _tokens.length; ++i) {


43        for (uint256 i; i < _pumps.length; ++i) {

```

## [G-12] Use solidity version 0.8.20 to gain some gas boost

Upgrade to the latest solidity version 0.8.20 to get additional gas savings. See latest release for reference: https://blog.soliditylang.org/2023/05/10/solidity-0.8.20-release-announcement/

https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol

```solidity
File: functions/ConstantProduct2.sol

3   pragma solidity ^0.8.17;

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ProportionalLPToken2.sol

```solidity
File: functions/ProportionalLPToken2.sol

3   pragma solidity ^0.8.17;

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol

```solidity
File: pumps/MultiFlowPump.sol

3   pragma solidity ^0.8.17;

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol

```solidity
File: src/Aquifer.sol

3   pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol

````solidity
File: src/Well.sol
```solidity

3   pragma solidity ^0.8.17;

````

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol

```solidity
File: libraries/LibWellConstructor.sol

4   pragma solidity ^0.8.17;

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol

```solidity
File: libraries/LibLastReserveBytes.sol

3   pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol

```solidity
File: libraries/LibContractInfo.sol


3   pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes16.sol

```solidity
File: libraries/LibBytes16.sol

3   pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol

```solidity
File: libraries/LibBytes.sol

3   pragma solidity ^0.8.17;
```

## [G-13] With assembly, ``.call (bool success)` transfer can be done gas-optimized

In Solidity, the transfer method is commonly used to send Ether from one address to another. However, the transfer method uses a fixed gas stipend of 2300 gas, which can result in out-of-gas errors if the recipient contract requires more gas to process the transaction.

https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L55

```solidity
File: src/Aquifer.sol

55   (bool success, bytes memory returnData) = well.call(initFunctionCall);
```

## [G-14] `x += y` costs more gas than `x = x + y` for state variables

In Solidity, using x = x + y instead of x += y can sometimes save gas when working with state variables.
The reason for this is that the Solidity compiler generates more optimized code for x = x + y than for x += y when working with state variables. The x += y operation requires an additional read operation to retrieve the current value of x before performing the addition, whereas x = x + y can be optimized to combine the read and write operations into a single step.

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L103

```solidity
File: src/Well.sol

103   dataLoc += PACKED_ADDRESS;
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L105

```solidity
File: src/Well.sol

105   dataLoc += ONE_WORD;
```

## [G-15] String literals passed to `abi.encode()/abi.encodePacked()` should not be split by commas

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L81

```solidity
File: libraries/LibWellConstructor.sol

81      initFunctionCall = abi.encodeWithSignature("init(string,string)", name, symbol);
```

## [G-16] Remove or replace unused variables

To save gas, you can remove any unused variables from your code. Additionally, you can also replace unused variables with other variables or constants to optimize storage usage.

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes16.sol#L14

```solidity
File: libraries/LibBytes16.sol

14      bytes32 private constant ZERO_BYTES = bytes32(0);
```

## [G-17] Using a positive conditional flow to save a `NOT` opcode

In Solidity, you can use a positive conditional flow to save a NOT opcode and optimize gas usage.
The NOT opcode is used to perform a bitwise negation operation on a value in Solidity. However, using a positive conditional flow can help avoid the need for a NOT opcode and reduce gas usage.

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L748

```solidity
File: src/Well.sol

748       if (!foundI) revert InvalidTokens();
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L749

```solidity
File: src/Well.sol

749       if (!foundJ) revert InvalidTokens();
```
