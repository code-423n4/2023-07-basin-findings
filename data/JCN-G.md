# Summary

A majority of the optimizations were benchmarked via the protocol's tests, i.e. using the following config: `solc version 0.8.17`, `optimizer on`, and `800 runs`. Optimizations that were not benchmarked are explained via EVM gas costs and opcodes. The following command was used to run the tests for the benchmarks below: `forge test --ffi --no-match-test invariant --gas-report`.

Below are the overall average gas savings for the following tested functions (with all the optimizations applied):

| Function |    Before   |    After   | Avg Gas Savings |
| ------ | -------- | -------- | ------- |
| Well.addLiquidity |  131592  |  131499 |  93 | 
| Well.addLiquidityFeeOnTransfer |  70773  |  70657 |  116 | 
| Well.getAddLiquidityOut |  13655  |  13597 |  58 | 
| Well.getRemoveLiquidityImbalancedIn |  11336  |  11277 |  59 | 
| Well.pumps |  3719  |  3619 |  100 | 
| Well.removeLiquidity |  35113  |  34937 |  176 | 
| Well.removeLiquidityImbalanced |  89489  |  89303 |  186 | 
| Well.shift |  38386  |  38220 |  166 | 
| Well.skim  |  53028  |  52970 |  58 | 
| Well.swapFrom |  100551  |  100380 |  171 | 
| Well.swapTo |  28400  |  28342 |  58 | 
| Well.sync |  16007  |  15891 |  116 | 
| MultiFlowPump.readCumulativeReserves |  47896  |  47838 |  58 | 
| MultiFlowPump.readInstantaneousReserves |  57767  |  57706 |  61 | 
| MultiFlowPump.readLastInstantaneousReserves |  12119  |  12058 |  61 | 
| MultiFlowPump.readLastReserves |  11426  |  11365 |  61 | 
| MultiFlowPump.update |  75406  |  75352 |  54 | 

**Total gas saved across all listed functions: 1652**

*Notes*: 

- The [Gas report](#gas-report-output-with-all-optimizations-applied) output, after all optimizations have been applied, can be found at the end of the report.
- The final diff for the contract, with all the optimizations applied, can be found [here](https://gist.github.com/0xJCN/746cdbf94783977df609fcbacd38dc54).
- The test suite uses fuzzing and this makes the gas usage in the `Avg` and `Max` columns of the gas report unpredictable. For this reason, the `Med` column is used for benchmarking.
- Optimizations to `view`/`pure` functions are done with the assumption that said functions will be called within state-mutating functions in contracts that integrate with Basin.

## Gas Optimizations
| Number |Issue|Instances|
|-|:-|:-:| 
| [G-01](#use-do-while-loops-instead-of-for-loops) | Use `do while` loops instead of `for loops` | 22 | 
| [G-02](#cache-memory-pointer-to-avoid-complex-offset-calculations-in-memory) | Cache memory pointer to avoid complex offset calculations in memory | 2 | 
| [G-03](#cache-memorycalldata-values-to-avoid-redundant-offset-calculations-to-read-the-same-data) | Cache memory/calldata values to avoid redundant offset calculations to read the same data | 2 | 
| [G-04](#refactor-function-to-avoid-potentially-iterating-through-array-twice) | Refactor function to avoid potentially iterating through array twice | 2 | 
| [G-05](#cache-external-call) | Cache External Call | 1 | 
| [G-06](#sort-array-offchain-to-check-duplicates-in-on-instead-of-on2) | Sort array offchain to check duplicates in O(n) instead of O(n^2) | - | 

## Use `do while` loops instead of `for loops`
A do while loop will cost less gas since the condition is not being checked for the first iteration. Other optimizations, such as caching length, precrementing, and using unchecked blocks are not included in the Diff since they are included in the bot race report.

*Note: The first 2 instances do not include benchmarks as those loops will only run in the case where the pump has never been used before. For this reason, and due to the fact that the `Avg` and `Max` gas usage for the `update` function changes unpredicatbly due to fuzzing, the `Med` column does not reflect any gas savings. The gas savings for these instances can therefore be extrapolated from the other instances presented.*

Total Instances: `22`

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L84-L87

```solidity
File: src/pumps/MultiFlowPump.sol
84:            for (uint256 i; i < numberOfReserves; ++i) {
85:                // If a reserve is 0, then the pump cannot be initialized.
86:                if (reserves[i] == 0) return;
87:            }
```
```diff
diff --git a/./src/pumps/MultiFlowPump.sol b/./src/pumps/MultiFlowPumpNew.sol
index e2aab5d..255001d 100644
--- a/./src/pumps/MultiFlowPump.sol
+++ b/./src/pumps/MultiFlowPumpNew.sol
@@ -81,10 +81,12 @@ contract MultiFlowPump is IPump, IMultiFlowPumpErrors, IInstantaneousPump, ICumu

         // If the last timestamp is 0, then the pump has never been used before.
         if (pumpState.lastTimestamp == 0) {
-            for (uint256 i; i < numberOfReserves; ++i) {
+            uint256 i;
+            do {
                 // If a reserve is 0, then the pump cannot be initialized.
                 if (reserves[i] == 0) return;
-            }
+                ++i;
+            } while (i < numberOfReserves);
             _init(slot, uint40(block.timestamp), reserves);
             return;
         }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L146-L155

```solidity
File: src/pumps/MultiFlowPump.sol
146:    function _init(bytes32 slot, uint40 lastTimestamp, uint256[] memory reserves) internal {
147:        uint256 numberOfReserves = reserves.length;
148:        bytes16[] memory byteReserves = new bytes16[](numberOfReserves);
149:
150:        // Skip {_capReserve} since we have no prior reference
151:
152:        for (uint256 i; i < numberOfReserves; ++i) {
153:            if (reserves[i] == 0) return;
154:            byteReserves[i] = reserves[i].fromUIntToLog2();
155:        }
```
```diff
diff --git a/src/pumps/MultiFlowPump.sol b/src/pumps/MultiFlowPump.sol
index e2aab5d..0aed1f5 100644
--- a/src/pumps/MultiFlowPump.sol
+++ b/src/pumps/MultiFlowPump.sol
@@ -148,11 +148,12 @@ contract MultiFlowPump is IPump, IMultiFlowPumpErrors, IInstantaneousPump, ICumu
         bytes16[] memory byteReserves = new bytes16[](numberOfReserves);

         // Skip {_capReserve} since we have no prior reference
-
-        for (uint256 i; i < numberOfReserves; ++i) {
+        uint256 i;
+        do {
             if (reserves[i] == 0) return;
             byteReserves[i] = reserves[i].fromUIntToLog2();
-        }
+            ++i;
+        } while (i < numberOfReserves);

         // Write: Last Timestamp & Last Reserves
         slot.storeLastReserves(lastTimestamp, byteReserves);
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L116-L125

*Gas Savings for `MultiFlowPump.update`, obtained via protocol's tests: Med 55 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  75407   |  175264  |  75263  |   30077  |
| After  |  75352   |  97147   |  75202  |   30077  |

```solidity
File: src/pumps/MultiFlowPump.sol
116:        for (uint256 i; i < numberOfReserves; ++i) {
117:            // Use a minimum of 1 for reserve. Geometric means will be set to 0 if a reserve is 0.
118:            pumpState.lastReserves[i] = _capReserve(
119:                pumpState.lastReserves[i], (reserves[i] > 0 ? reserves[i] : 1).fromUIntToLog2(), blocksPassed
120:            );
121:            pumpState.emaReserves[i] =
122:                pumpState.lastReserves[i].mul((ABDKMathQuad.ONE.sub(alphaN))).add(pumpState.emaReserves[i].mul(alphaN));
123:            pumpState.cumulativeReserves[i] =
124:                pumpState.cumulativeReserves[i].add(pumpState.lastReserves[i].mul(deltaTimestampBytes));
125:        }
```
```diff
diff --git a/src/pumps/MultiFlowPump.sol b/src/pumps/MultiFlowPump.sol
index e2aab5d..2a5ceea 100644
--- a/src/pumps/MultiFlowPump.sol
+++ b/src/pumps/MultiFlowPump.sol
@@ -112,8 +112,9 @@ contract MultiFlowPump is IPump, IMultiFlowPumpErrors, IInstantaneousPump, ICumu
             // Relies on the assumption that a block can only occur every `BLOCK_TIME` seconds.
             blocksPassed = (deltaTimestamp / BLOCK_TIME).fromUInt();
         }
-
-        for (uint256 i; i < numberOfReserves; ++i) {
+
+        uint256 i;
+        do {
             // Use a minimum of 1 for reserve. Geometric means will be set to 0 if a reserve is 0.
             pumpState.lastReserves[i] = _capReserve(
                 pumpState.lastReserves[i], (reserves[i] > 0 ? reserves[i] : 1).fromUIntToLog2(), blocksPassed
@@ -122,7 +123,8 @@ contract MultiFlowPump is IPump, IMultiFlowPumpErrors, IInstantaneousPump, ICumu
                 pumpState.lastReserves[i].mul((ABDKMathQuad.ONE.sub(alphaN))).add(pumpState.emaReserves[i].mul(alphaN));
             pumpState.cumulativeReserves[i] =
                 pumpState.cumulativeReserves[i].add(pumpState.lastReserves[i].mul(deltaTimestampBytes));
-        }
+            ++i;
+        } while (i < numberOfReserves);

         // Write: Cumulative & EMA Reserves
         // Order matters: work backwards to avoid using a new memory var to count up
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L171-L180

*Gas Savings for `MultiFlowPump.readLastReserves`, obtained via protocol's tests: Med 61 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  11482   |  22638   |  11799  |     7    |
| After  |  11421   |  28771   |  12632  |     7    |

```solidity
File: src/pumps/MultiFlowPump.sol
171:    function readLastReserves(address well) public view returns (uint256[] memory reserves) {
172:        bytes32 slot = _getSlotForAddress(well);
173:        (uint8 numberOfReserves,, bytes16[] memory bytesReserves) = slot.readLastReserves();
174:        if (numberOfReserves == 0) {
175:            revert NotInitialized();
176:        }
177:        reserves = new uint256[](numberOfReserves);
178:        for (uint256 i; i < numberOfReserves; ++i) {
179:            reserves[i] = bytesReserves[i].pow_2ToUInt();
180:        }
```
```diff
diff --git a/src/pumps/MultiFlowPump.sol b/src/pumps/MultiFlowPump.sol
index e2aab5d..40f58e0 100644
--- a/src/pumps/MultiFlowPump.sol
+++ b/src/pumps/MultiFlowPump.sol
@@ -175,9 +175,11 @@ contract MultiFlowPump is IPump, IMultiFlowPumpErrors, IInstantaneousPump, ICumu
             revert NotInitialized();
         }
         reserves = new uint256[](numberOfReserves);
-        for (uint256 i; i < numberOfReserves; ++i) {
+        uint256 i;
+        do {
             reserves[i] = bytesReserves[i].pow_2ToUInt();
-        }
+            ++i;
+        } while (i < numberOfReserves);
     }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L222-L236

*Gas Savings for `MultiFlowPump.readLastInstantaneousReserves`, obtained via protocol's tests: Med 61 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  12119   |  14273   |  10649  |     5    |
| After  |  12058   |  14212   |  10601  |     5    |

```solidity
File: src/pumps/MultiFlowPump.sol
222:    function readLastInstantaneousReserves(address well) public view returns (uint256[] memory reserves) {
223:        bytes32 slot = _getSlotForAddress(well);
224:        uint8 numberOfReserves = slot.readNumberOfReserves();
225:        if (numberOfReserves == 0) {
226:            revert NotInitialized();
227:        }
228:        uint256 offset = _getSlotsOffset(numberOfReserves);
229:        assembly {
230:            slot := add(slot, offset)
231:        }
232:        bytes16[] memory byteReserves = slot.readBytes16(numberOfReserves);
233:        reserves = new uint256[](numberOfReserves);
234:        for (uint256 i; i < numberOfReserves; ++i) {
235:            reserves[i] = byteReserves[i].pow_2ToUInt();
236:        }
```
```diff
diff --git a/src/pumps/MultiFlowPump.sol b/src/pumps/MultiFlowPump.sol
index e2aab5d..0abb39f 100644
--- a/src/pumps/MultiFlowPump.sol
+++ b/src/pumps/MultiFlowPump.sol
@@ -231,9 +231,11 @@ contract MultiFlowPump is IPump, IMultiFlowPumpErrors, IInstantaneousPump, ICumu
         }
         bytes16[] memory byteReserves = slot.readBytes16(numberOfReserves);
         reserves = new uint256[](numberOfReserves);
-        for (uint256 i; i < numberOfReserves; ++i) {
+        uint256 i;
+        do {
             reserves[i] = byteReserves[i].pow_2ToUInt();
-        }
+            ++i;
+        } while (i < numberOfReserves);
     }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L239-L259

*Gas Savings for `MultiFlowPump.readInstantaneousReserves`, obtained via protocol's tests: Med 61 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  57767   |  65551   |  53634  |     7    |
| After  |  57706   |  65490   |  53582  |     7    |

```solidity
src/pumps/MultiFlowPump.sol
239:    function readInstantaneousReserves(address well, bytes memory) public view returns (uint256[] memory emaReserves) {
...
255:        for (uint256 i; i < numberOfReserves; ++i) {
256:            lastReserves[i] = _capReserve(lastReserves[i], reserves[i].fromUIntToLog2(), blocksPassed);
257:            emaReserves[i] =
258:                lastReserves[i].mul((ABDKMathQuad.ONE.sub(alphaN))).add(lastEmaReserves[i].mul(alphaN)).pow_2ToUInt();
259:        }
```
```diff
diff --git a/src/pumps/MultiFlowPump.sol b/src/pumps/MultiFlowPump.sol
index e2aab5d..ecd2a84 100644
--- a/src/pumps/MultiFlowPump.sol
+++ b/src/pumps/MultiFlowPump.sol
@@ -252,11 +252,13 @@ contract MultiFlowPump is IPump, IMultiFlowPumpErrors, IInstantaneousPump, ICumu
         bytes16 blocksPassed = (deltaTimestamp / BLOCK_TIME).fromUInt();
         bytes16 alphaN = ALPHA.powu(deltaTimestamp);
         emaReserves = new uint256[](numberOfReserves);
-        for (uint256 i; i < numberOfReserves; ++i) {
+        uint256 i;
+        do {
             lastReserves[i] = _capReserve(lastReserves[i], reserves[i].fromUIntToLog2(), blocksPassed);
             emaReserves[i] =
                 lastReserves[i].mul((ABDKMathQuad.ONE.sub(alphaN))).add(lastEmaReserves[i].mul(alphaN)).pow_2ToUInt();
-        }
+            ++i;
+        } while (i < numberOfReserves);
     }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L285-L304

*Gas Savings for `MultiFlowPump.readCumulativeReserves`, obtained via protocol's tests: Med 58 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  48566   |  91220   |  51819  |    17    |
| After  |  48508   |  70230   |  50534  |    17    |

```solidity
File: src/pumps/MultiFlowPump.sol
285:    function _readCumulativeReserves(address well) internal view returns (bytes16[] memory cumulativeReserves) {
...
301:        for (uint256 i; i < cumulativeReserves.length; ++i) {
302:            lastReserves[i] = _capReserve(lastReserves[i], reserves[i].fromUIntToLog2(), blocksPassed);
303:            cumulativeReserves[i] = cumulativeReserves[i].add(lastReserves[i].mul(deltaTimestampBytes));
304:        }
```
```diff
diff --git a/src/pumps/MultiFlowPump.sol b/src/pumps/MultiFlowPump.sol
index e2aab5d..23a0978 100644
--- a/src/pumps/MultiFlowPump.sol
+++ b/src/pumps/MultiFlowPump.sol
@@ -298,10 +298,12 @@ contract MultiFlowPump is IPump, IMultiFlowPumpErrors, IInstantaneousPump, ICumu
         bytes16 deltaTimestampBytes = deltaTimestamp.fromUInt();
         bytes16 blocksPassed = (deltaTimestamp / BLOCK_TIME).fromUInt();
         // Currently, there is so support for overflow.
-        for (uint256 i; i < cumulativeReserves.length; ++i) {
+        uint256 i;
+        do {
             lastReserves[i] = _capReserve(lastReserves[i], reserves[i].fromUIntToLog2(), blocksPassed);
             cumulativeReserves[i] = cumulativeReserves[i].add(lastReserves[i].mul(deltaTimestampBytes));
-        }
+            ++i;
+        } while (i < cumulativeReserves.length);
     }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L307-L326

*Gas Savings for `MultiFlowPump.readTwaReserves`, obtained via protocol's tests: Med 58 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  62379   |  115540  |  60298  |    12    |
| After  |  62321   |  143741  |  62610  |    12    |

```solidity
File: src/pumps/MultiFlowPump.sol
307:    function readTwaReserves(
308:        address well,
309:        bytes calldata startCumulativeReserves,
310:        uint256 startTimestamp,
311:        bytes memory
312:    ) public view returns (uint256[] memory twaReserves, bytes memory cumulativeReserves) {
...
322:        for (uint256 i; i < byteCumulativeReserves.length; ++i) {
323:            // Currently, there is no support for overflow.
324:            twaReserves[i] =
325:                (byteCumulativeReserves[i].sub(byteStartCumulativeReserves[i])).div(deltaTimestamp).pow_2ToUInt();
326:        }
```
```diff
diff --git a/src/pumps/MultiFlowPump.sol b/src/pumps/MultiFlowPump.sol
index e2aab5d..6ddb60f 100644
--- a/src/pumps/MultiFlowPump.sol
+++ b/src/pumps/MultiFlowPump.sol
@@ -319,11 +319,13 @@ contract MultiFlowPump is IPump, IMultiFlowPumpErrors, IInstantaneousPump, ICumu
         if (deltaTimestamp == bytes16(0)) {
             revert NoTimePassed();
         }
-        for (uint256 i; i < byteCumulativeReserves.length; ++i) {
+        uint256 i;
+        do {
             // Currently, there is no support for overflow.
             twaReserves[i] =
                 (byteCumulativeReserves[i].sub(byteStartCumulativeReserves[i])).div(deltaTimestamp).pow_2ToUInt();
-        }
+            ++i;
+        } while (i < byteCumulativeReserves.length);
         cumulativeReserves = abi.encode(byteCumulativeReserves);
     }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L94-L108

*Gas Savings for `Well.pumps`, obtained via protocol's tests: Med 49 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  3719    |  7773    |  3969   |    117   |
| After  |  3670    |  5535    |  3856   |    117   |

```solidity
File: src/Well.sol
94:    function pumps() public pure returns (Call[] memory _pumps) {
...
101:        for (uint256 i; i < _pumps.length; i++) {
102:            _pumps[i].target = _getArgAddress(dataLoc);
103:            dataLoc += PACKED_ADDRESS;
104:            pumpDataLength = _getArgUint256(dataLoc);
105:            dataLoc += ONE_WORD;
106:            _pumps[i].data = _getArgBytes(dataLoc, pumpDataLength);
107:            dataLoc += pumpDataLength;
108:        }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..8b0dc95 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -98,14 +98,16 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
         uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD + wellFunctionDataLength();

         uint256 pumpDataLength;
-        for (uint256 i; i < _pumps.length; i++) {
+        uint256 i;
+        do {
             _pumps[i].target = _getArgAddress(dataLoc);
             dataLoc += PACKED_ADDRESS;
             pumpDataLength = _getArgUint256(dataLoc);
             dataLoc += ONE_WORD;
             _pumps[i].data = _getArgBytes(dataLoc, pumpDataLength);
             dataLoc += pumpDataLength;
-        }
+            ++i;
+        } while (i < _pumps.length);
     }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L352-L365

*Gas Savings for `Well.shift`, obtained via protocol's tests: Med 58 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  38386   |  59296   |  36740  |    6     |
| After  |  38328   |  59238   |  36681  |    6     |

```solidity
File: src/Well.sol
352:    function shift(
353:        IERC20 tokenOut,
354:        uint256 minAmountOut,
355:        address recipient
356:    ) external nonReentrant returns (uint256 amountOut) {
...
363:        for (uint256 i; i < _tokens.length; ++i) {
364:            reserves[i] = _tokens[i].balanceOf(address(this));
365:        }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..dd0855a 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -360,9 +360,11 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
         // Use the balances of the pool instead of the stored reserves.
         // If there is a change in token balances relative to the currently
         // stored reserves, the extra tokens can be shifted into `tokenOut`.
-        for (uint256 i; i < _tokens.length; ++i) {
+        uint256 i;
+        do {
             reserves[i] = _tokens[i].balanceOf(address(this));
-        }
+            ++i;
+        } while (i < _tokens.length);
         uint256 j = _getJ(_tokens, tokenOut);
         amountOut = reserves[j] - _calcReserve(wellFunction(), reserves, j, totalSupply());
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L379-L384

*Gas Savings for `Well.getShiftOut`, obtained via protocol's tests: Med 58 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  11359   |  11359   |  11289  |    3     |
| After  |  11301   |  11301   |  11231  |    3     |

```solidity
File: src/Well.sol
379:    function getShiftOut(IERC20 tokenOut) external view returns (uint256 amountOut) {
380:        IERC20[] memory _tokens = tokens();
381:        uint256[] memory reserves = new uint256[](_tokens.length);
382:        for (uint256 i; i < _tokens.length; ++i) {
383:            reserves[i] = _tokens[i].balanceOf(address(this));
384:        }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..b993a2c 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -379,9 +379,11 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
     function getShiftOut(IERC20 tokenOut) external view returns (uint256 amountOut) {
         IERC20[] memory _tokens = tokens();
         uint256[] memory reserves = new uint256[](_tokens.length);
-        for (uint256 i; i < _tokens.length; ++i) {
+        uint256 i;
+        do {
             reserves[i] = _tokens[i].balanceOf(address(this));
-        }
+            ++i;
+        } while (i < _tokens.length);

         uint256 j = _getJ(_tokens, tokenOut);
         amountOut = reserves[j] - _calcReserve(wellFunction(), reserves, j, totalSupply());
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L413-L433

*Gas Savings for `Well.addLiquidity`, obtained via protocol's tests: Med 47 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  131592  |  198716  |  129836 |    140   |
| After  |  131545  |  198648  |  129787 |    140   |

*Gas Savings for `Well.addLiquidityFeeOnTransfer`, obtained via protocol's tests: Med 58 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  70773   |  109514  |  61738  |    13    |
| After  |  70715   |  109456  |  61692  |    13    |

```solidity
File: src/Well.sol
413:    function _addLiquidity(
414:        uint256[] memory tokenAmountsIn,
415:        uint256 minLpAmountOut,
416:        address recipient,
417:        bool feeOnTransfer
418:    ) internal returns (uint256 lpAmountOut) {
...
422:        if (feeOnTransfer) {
423:            for (uint256 i; i < _tokens.length; ++i) {
424:                if (tokenAmountsIn[i] == 0) continue;
425:                tokenAmountsIn[i] = _safeTransferFromFeeOnTransfer(_tokens[i], msg.sender, tokenAmountsIn[i]);
426:                reserves[i] = reserves[i] + tokenAmountsIn[i];
427:            }
428:        } else {
429:            for (uint256 i; i < _tokens.length; ++i) {
430:                if (tokenAmountsIn[i] == 0) continue;
431:                _tokens[i].safeTransferFrom(msg.sender, address(this), tokenAmountsIn[i]);
432:                reserves[i] = reserves[i] + tokenAmountsIn[i];
433:            }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..d0eeb98 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -420,17 +420,27 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
         uint256[] memory reserves = _updatePumps(_tokens.length);

         if (feeOnTransfer) {
-            for (uint256 i; i < _tokens.length; ++i) {
-                if (tokenAmountsIn[i] == 0) continue;
+            uint256 i;
+            do {
+                if (tokenAmountsIn[i] == 0) {
+                    ++i;
+                    continue;
+                }
                 tokenAmountsIn[i] = _safeTransferFromFeeOnTransfer(_tokens[i], msg.sender, tokenAmountsIn[i]);
                 reserves[i] = reserves[i] + tokenAmountsIn[i];
-            }
+                ++i;
+            } while (i < _tokens.length);
         } else {
-            for (uint256 i; i < _tokens.length; ++i) {
-                if (tokenAmountsIn[i] == 0) continue;
+            uint256 i;
+            do {
+                if (tokenAmountsIn[i] == 0) {
+                    ++i;
+                    continue;
+                }
                 _tokens[i].safeTransferFrom(msg.sender, address(this), tokenAmountsIn[i]);
                 reserves[i] = reserves[i] + tokenAmountsIn[i];
-            }
+                ++i;
+            } while (i < _tokens.length);
         }

         lpAmountOut = _calcLpTokenSupply(wellFunction(), reserves) - totalSupply();
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L449-L454

*Gas Savings for `Well.getAddLiquidityOut`, obtained via protocol's tests: Med 58 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  13655   |  13655   |  12507  |    17    |
| After  |  13597   |  13621   |  12452  |    17    |

```solidity
File: src/Well.sol
449:    function getAddLiquidityOut(uint256[] memory tokenAmountsIn) external view returns (uint256 lpAmountOut) {
450:        IERC20[] memory _tokens = tokens();
451:        uint256[] memory reserves = _getReserves(_tokens.length);
452:        for (uint256 i; i < _tokens.length; ++i) {
453:            reserves[i] = reserves[i] + tokenAmountsIn[i];
454:        }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..414c7c4 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -449,9 +449,11 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
     function getAddLiquidityOut(uint256[] memory tokenAmountsIn) external view returns (uint256 lpAmountOut) {
         IERC20[] memory _tokens = tokens();
         uint256[] memory reserves = _getReserves(_tokens.length);
-        for (uint256 i; i < _tokens.length; ++i) {
+        uint256 i;
+        do {
             reserves[i] = reserves[i] + tokenAmountsIn[i];
-        }
+            ++i;
+        } while (i < _tokens.length);
         lpAmountOut = _calcLpTokenSupply(wellFunction(), reserves) - totalSupply();
     }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L460-L479

*Gas Savings for `Well.removeLiquidity`, obtained via protocol's tests: Med 43 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  55013   |  119552  |  58246  |    8     |
| After  |  54970   |  119494  |  58202  |    8     |

```solidity
File: src/Well.sol
460:    function removeLiquidity(
461:        uint256 lpAmountIn,
462:        uint256[] calldata minTokenAmountsOut,
463:        address recipient,
464:        uint256 deadline
465:    ) external nonReentrant expire(deadline) returns (uint256[] memory tokenAmountsOut) {
...
473:        for (uint256 i; i < _tokens.length; ++i) {
474:            if (tokenAmountsOut[i] < minTokenAmountsOut[i]) {
475:                revert SlippageOut(tokenAmountsOut[i], minTokenAmountsOut[i]);
476:            }
477:            _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);
478:            reserves[i] = reserves[i] - tokenAmountsOut[i];
479:        }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..9957f95 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -470,13 +470,15 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
         tokenAmountsOut = new uint256[](_tokens.length);
         _burn(msg.sender, lpAmountIn);
         tokenAmountsOut = _calcLPTokenUnderlying(wellFunction(), lpAmountIn, reserves, lpTokenSupply);
-        for (uint256 i; i < _tokens.length; ++i) {
+        uint256 i;
+        do {
             if (tokenAmountsOut[i] < minTokenAmountsOut[i]) {
                 revert SlippageOut(tokenAmountsOut[i], minTokenAmountsOut[i]);
             }
             _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);
             reserves[i] = reserves[i] - tokenAmountsOut[i];
-        }
+            ++i;
+        } while (i < _tokens.length);

         _setReserves(_tokens, reserves);
         emit RemoveLiquidity(lpAmountIn, tokenAmountsOut, recipient);
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L548-L560

*Gas Savings for `Well.removeLiquidityImbalanced`, obtained via protocol's tests: Med 58 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  89489   |  97989   |  72642  |    5     |
| After  |  89431   |  97931   |  72595  |    5     |

```solidity
File: src/Well.sol
548:    function removeLiquidityImbalanced(
549:        uint256 maxLpAmountIn,
550:        uint256[] calldata tokenAmountsOut,
551:        address recipient,
552:        uint256 deadline
553:    ) external nonReentrant expire(deadline) returns (uint256 lpAmountIn) {
...
557:        for (uint256 i; i < _tokens.length; ++i) {
558:            _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);
559:            reserves[i] = reserves[i] - tokenAmountsOut[i];
560:        }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..256c7d2 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -553,11 +553,13 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
     ) external nonReentrant expire(deadline) returns (uint256 lpAmountIn) {
         IERC20[] memory _tokens = tokens();
         uint256[] memory reserves = _updatePumps(_tokens.length);
-
-        for (uint256 i; i < _tokens.length; ++i) {
+
+        uint256 i;
+        do {
             _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);
             reserves[i] = reserves[i] - tokenAmountsOut[i];
-        }
+            ++i;
+        } while (i < _tokens.length);

         lpAmountIn = totalSupply() - _calcLpTokenSupply(wellFunction(), reserves);
         if (lpAmountIn > maxLpAmountIn) {
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L572-L581

*Gas Savings for `Well.getRemoveLiquidityImbalancedIn`, obtained via protocol's tests: Med 59 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  11336   |  13336   |  11236  |    5     |
| After  |  11277   |  13277   |  11177  |    5     |

```solidity
File: src/Well.sol
572:    function getRemoveLiquidityImbalancedIn(uint256[] calldata tokenAmountsOut)
573:        external
575:        view
576:        returns (uint256 lpAmountIn)
577:    {
578:        IERC20[] memory _tokens = tokens();
579:        uint256[] memory reserves = _getReserves(_tokens.length);
580:        for (uint256 i; i < _tokens.length; ++i) {
581:            reserves[i] = reserves[i] - tokenAmountsOut[i];
582:        }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..5995894 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -576,9 +576,11 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
     {
         IERC20[] memory _tokens = tokens();
         uint256[] memory reserves = _getReserves(_tokens.length);
-        for (uint256 i; i < _tokens.length; ++i) {
+        uint256 i;
+        do {
             reserves[i] = reserves[i] - tokenAmountsOut[i];
-        }
+            ++i;
+        } while (i < _tokens.length);
         lpAmountIn = totalSupply() - _calcLpTokenSupply(wellFunction(), reserves);
     }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L590-L595

*Gas Savings for `Well.sync`, obtained via protocol's tests: Med 58 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  16007   |  16007   |  16007  |    5     |
| After  |  15949   |  15949   |  15949  |    5     |

```solidity
File: src/Well.sol
590:    function sync() external nonReentrant {
591:        IERC20[] memory _tokens = tokens();
592:        uint256[] memory reserves = new uint256[](_tokens.length);
593:        for (uint256 i; i < _tokens.length; ++i) {
594:            reserves[i] = _tokens[i].balanceOf(address(this));
595:        }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..83e7a28 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -590,9 +590,11 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
     function sync() external nonReentrant {
         IERC20[] memory _tokens = tokens();
         uint256[] memory reserves = new uint256[](_tokens.length);
-        for (uint256 i; i < _tokens.length; ++i) {
+        uint256 i;
+        do {
             reserves[i] = _tokens[i].balanceOf(address(this));
-        }
+            ++i;
+        } while (i < _tokens.length);
         _setReserves(_tokens, reserves);
         emit Sync(reserves);
     }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L603-L612

*Gas Savings for `Well.skim`, obtained via protocol's tests: Med 58 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  53028   |  53028   |  53028  |    1     |
| After  |  52970   |  52970   |  52970  |    1     |

```solidity
File: src/Well.sol
603:    function skim(address recipient) external nonReentrant returns (uint256[] memory skimAmounts) {
604:        IERC20[] memory _tokens = tokens();
605:        uint256[] memory reserves = _getReserves(_tokens.length);
606:        skimAmounts = new uint256[](_tokens.length);
607:        for (uint256 i; i < _tokens.length; ++i) {
608:            skimAmounts[i] = _tokens[i].balanceOf(address(this)) - reserves[i];
609:            if (skimAmounts[i] > 0) {
610:                _tokens[i].safeTransfer(recipient, skimAmounts[i]);
611:            }
612:        }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..d5aeb15 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -604,12 +604,14 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
         IERC20[] memory _tokens = tokens();
         uint256[] memory reserves = _getReserves(_tokens.length);
         skimAmounts = new uint256[](_tokens.length);
-        for (uint256 i; i < _tokens.length; ++i) {
+        uint256 i;
+        do {
             skimAmounts[i] = _tokens[i].balanceOf(address(this)) - reserves[i];
             if (skimAmounts[i] > 0) {
                 _tokens[i].safeTransfer(recipient, skimAmounts[i]);
             }
-        }
+            ++i;
+        } while (i < _tokens.length);
     }

     function getReserves() external view returns (uint256[] memory reserves) {
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L632-L637

*Gas Savings for `Well.swapFrom`, obtained via protocol's tests: Med 58 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  100551  |  151312  |  88489  |    35    |
| After  |  100493  |  151254  |  88454  |    35    |

```solidity
File: src/Well.sol
632:    function _setReserves(IERC20[] memory _tokens, uint256[] memory reserves) internal {
633:        for (uint256 i; i < reserves.length; ++i) {
634:            if (reserves[i] > _tokens[i].balanceOf(address(this))) revert InvalidReserves();
635:        }
636:        LibBytes.storeUint128(RESERVES_STORAGE_SLOT, reserves);
637:    }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..491b4f2 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -630,9 +630,11 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
      * sets the Well's reserves of each token by writing to byte storage.
      */
     function _setReserves(IERC20[] memory _tokens, uint256[] memory reserves) internal {
-        for (uint256 i; i < reserves.length; ++i) {
+        uint256 i;
+        do {
             if (reserves[i] > _tokens[i].balanceOf(address(this))) revert InvalidReserves();
-        }
+            ++i;
+        } while (i < reserves.length);
         LibBytes.storeUint128(RESERVES_STORAGE_SLOT, reserves);
     }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L645-L668

*Gas Savings for `Well.swapTo`, obtained via protocol's tests: Max 59 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  28400   |  89399   |  41401  |    7     |
| After  |  28400   |  89340   |  41393  |    7     |

```solidity
File: src/Well.sol
645:    function _updatePumps(uint256 _numberOfTokens) internal returns (uint256[] memory reserves) {
...
660:        } else {
661:            Call[] memory _pumps = pumps();
662:            for (uint256 i; i < _pumps.length; ++i) {
663:                // Don't revert if the update call fails.
664:                try IPump(_pumps[i].target).update(reserves, _pumps[i].data) {}
665:                catch {
666:                    // ignore reversion. If an external shutoff mechanism is added to a Pump, it could be called here.
667:                }
668:            }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..f16e41f 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -659,13 +659,15 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
             }
         } else {
             Call[] memory _pumps = pumps();
-            for (uint256 i; i < _pumps.length; ++i) {
+            uint256 i;
+            do {
                 // Don't revert if the update call fails.
                 try IPump(_pumps[i].target).update(reserves, _pumps[i].data) {}
                 catch {
                     // ignore reversion. If an external shutoff mechanism is added to a Pump, it could be called here.
                 }
-            }
+                ++i;
+            } while (i < _pumps.length);
         }
     }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L730-L746

*Gas Savings for `Well.swapTo`, obtained via protocol's tests: Med 58 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  28400   |  89399   |  41401  |    7     |
| After  |  28342   |  89341   |  41353  |    7     |

```solidity
File: src/Well.sol
730:    function _getIJ(
731:        IERC20[] memory _tokens,
732:        IERC20 iToken,
733:        IERC20 jToken
734:    ) internal pure returns (uint256 i, uint256 j) {
...
738:        for (uint256 k; k < _tokens.length; ++k) {
739:            if (iToken == _tokens[k]) {
740:                i = k;
741:                foundI = true;
742:            } else if (jToken == _tokens[k]) {
743:                j = k;
744:                foundJ = true;
745:            }
746:        }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..6cc4bcd 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -734,8 +734,9 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
     ) internal pure returns (uint256 i, uint256 j) {
         bool foundI = false;
         bool foundJ = false;
-
-        for (uint256 k; k < _tokens.length; ++k) {
+
+        uint256 k;
+        do {
             if (iToken == _tokens[k]) {
                 i = k;
                 foundI = true;
@@ -743,7 +744,8 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
                 j = k;
                 foundJ = true;
             }
-        }
+            ++k;
+        } while (k < _tokens.length);

         if (!foundI) revert InvalidTokens();
         if (!foundJ) revert InvalidTokens();
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L759-L766

*Gas Savings for `Well.shift`, obtained via protocol's tests: Med 20 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  38386   |  59296   |  36740  |    6     |
| After  |  38366   |  59268   |  36717  |    6     |

```solidity
File: src/Well.sol
759:    function _getJ(IERC20[] memory _tokens, IERC20 jToken) internal pure returns (uint256 j) {
760:        for (j; j < _tokens.length; ++j) {
761:            if (jToken == _tokens[j]) {
762:                return j;
763:            }
764:        }
765:        revert InvalidTokens();
766:    }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..eb22167 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -757,11 +757,13 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
      * the first one. A {Well} with duplicate tokens has been misconfigured.
      */
     function _getJ(IERC20[] memory _tokens, IERC20 jToken) internal pure returns (uint256 j) {
-        for (j; j < _tokens.length; ++j) {
+        uint256 j;
+        do {
             if (jToken == _tokens[j]) {
                 return j;
             }
-        }
+            ++j;
+        } while (j < _tokens.length);
         revert InvalidTokens();
     }
```

## Cache memory pointer to avoid complex offset calculations in memory
The memory variables in the following instances are complex types (i.e. arrays which contain structs) and thus will result in more complex offset calculations to retrieve specific data from memory. We can avoid peforming some of these offset calculations by instantiating memory pointers.

Total Instances: `2`

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L94-L108

*Gas Savings for `Well.pumps`, obtained via protocol's tests: Med 48 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  3719    |  7773    |  3969   |    117   |
| Before |  3671    |  5507    |  3826   |    117   |

### Cache memory pointer for `_pumps[i]`
```solidity
File: src/Well.sol
94:    function pumps() public pure returns (Call[] memory _pumps) {
...
97:        _pumps = new Call[](numberOfPumps());
...
101:        for (uint256 i; i < _pumps.length; i++) {
102:            _pumps[i].target = _getArgAddress(dataLoc);
103:            dataLoc += PACKED_ADDRESS;
104:            pumpDataLength = _getArgUint256(dataLoc);
105:            dataLoc += ONE_WORD;
106:            _pumps[i].data = _getArgBytes(dataLoc, pumpDataLength);
107:            dataLoc += pumpDataLength;
108:        }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..cb26e26 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -99,11 +99,12 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr

         uint256 pumpDataLength;
         for (uint256 i; i < _pumps.length; i++) {
-            _pumps[i].target = _getArgAddress(dataLoc);
+            Call memory _pump = _pumps[i];
+            _pump.target = _getArgAddress(dataLoc);
             dataLoc += PACKED_ADDRESS;
             pumpDataLength = _getArgUint256(dataLoc);
             dataLoc += ONE_WORD;
-            _pumps[i].data = _getArgBytes(dataLoc, pumpDataLength);
+            _pump.data = _getArgBytes(dataLoc, pumpDataLength);
             dataLoc += pumpDataLength;
         }
     }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L645-L668

*Gas Savings for `Well.swapTo`, obtained via protocol's tests: Max 72 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  28400   |  89399   |  41401  |    7     |
| After  |  28400   |  89327   |  41391  |    7     |

### Cache memory pointer for `_pumps[i]`
```solidity
File: src/Well.sol
645:    function _updatePumps(uint256 _numberOfTokens) internal returns (uint256[] memory reserves) {
...
660:        } else {
661:            Call[] memory _pumps = pumps();
662:            for (uint256 i; i < _pumps.length; ++i) {
663:                // Don't revert if the update call fails.
664:                try IPump(_pumps[i].target).update(reserves, _pumps[i].data) {}
665:                catch {
666:                    // ignore reversion. If an external shutoff mechanism is added to a Pump, it could be called here.
667:                }
668:            }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..08ac4eb 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -660,8 +660,9 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
         } else {
             Call[] memory _pumps = pumps();
             for (uint256 i; i < _pumps.length; ++i) {
+                Call memory _pump = _pumps[i];
                 // Don't revert if the update call fails.
-                try IPump(_pumps[i].target).update(reserves, _pumps[i].data) {}
+                try IPump(_pump.target).update(reserves, _pump.data) {}
                 catch {
                     // ignore reversion. If an external shutoff mechanism is added to a Pump, it could be called here.
                 }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L36-L50

*Note: The instance below is refers to a library function, and is not explicitly tested in the test suite. This optimization assumes that this library/function will be used within state-mutating functions within contracts that integrate with Basin. For these reasons this instance is not included in the final Diffs, but is included here for completeness.*

### Cache memory pointer for `_pumps[i]`
```solidity
File: src/libraries/LibWellConstructor.sol
36:    function encodeWellImmutableData(
37:        address _aquifer,
38:        IERC20[] memory _tokens,
39:        Call memory _wellFunction,
40:        Call[] memory _pumps
41:    ) internal pure returns (bytes memory immutableData) {
42:        bytes memory packedPumps;
43:        for (uint256 i; i < _pumps.length; ++i) {
44:            packedPumps = abi.encodePacked(
45:                packedPumps,            // previously packed pumps
46:                _pumps[i].target,       // pump address
47:                _pumps[i].data.length,  // pump data length
48:                _pumps[i].data          // pump data (bytes)
49:            );
50:        }
```
```diff
diff --git a/src/libraries/LibWellConstructor.sol b/src/libraries/LibWellConstructor.sol
index 507f2d9..c6b8cfa 100644
--- a/src/libraries/LibWellConstructor.sol
+++ b/src/libraries/LibWellConstructor.sol
@@ -41,11 +41,12 @@ library LibWellConstructor {
     ) internal pure returns (bytes memory immutableData) {
         bytes memory packedPumps;
         for (uint256 i; i < _pumps.length; ++i) {
+            Call memory _pump = _pumps[i];
             packedPumps = abi.encodePacked(
                 packedPumps,            // previously packed pumps
-                _pumps[i].target,       // pump address
-                _pumps[i].data.length,  // pump data length
-                _pumps[i].data          // pump data (bytes)
+                _pump.target,       // pump address
+                _pump.data.length,  // pump data length
+                _pump.data          // pump data (bytes)
             );
         }
```


## Cache `memory`/`calldata` values to avoid redundant offset calculations to read the same data
In the instances below, the same values from `memory`/`calldata` are being accessed multiple times. When the values are accessed, calculations are done to find their offset in `memory`/`calldata`. We can avoid performing these calculations multiple times by caching the value.

Total Instances: `2`

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L460-L479

*Gas Savings for `Well.removeLiquidity`, obtained via protocol's tests: Med 104 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  55013   |  119552  |  58246  |    8     |
| After  |  54909   |  119382  |  58122  |    8     |

### Can cache `tokenAmountsOut[i]` since it is accessed mutliple times
```solidity
File: src/Well.sol
460:    function removeLiquidity(
461:        uint256 lpAmountIn,
462:        uint256[] calldata minTokenAmountsOut,
463:        address recipient,
464:        uint256 deadline
465:    ) external nonReentrant expire(deadline) returns (uint256[] memory tokenAmountsOut) {
...
470:        tokenAmountsOut = new uint256[](_tokens.length);
471:        _burn(msg.sender, lpAmountIn);
472:        tokenAmountsOut = _calcLPTokenUnderlying(wellFunction(), lpAmountIn, reserves, lpTokenSupply);
473:        for (uint256 i; i < _tokens.length; ++i) {
474:            if (tokenAmountsOut[i] < minTokenAmountsOut[i]) {
475:                revert SlippageOut(tokenAmountsOut[i], minTokenAmountsOut[i]);
476:            }
477:            _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);
478:            reserves[i] = reserves[i] - tokenAmountsOut[i];
479:        }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..ceb1a3a 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -471,11 +471,12 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
         _burn(msg.sender, lpAmountIn);
         tokenAmountsOut = _calcLPTokenUnderlying(wellFunction(), lpAmountIn, reserves, lpTokenSupply);
         for (uint256 i; i < _tokens.length; ++i) {
-            if (tokenAmountsOut[i] < minTokenAmountsOut[i]) {
-                revert SlippageOut(tokenAmountsOut[i], minTokenAmountsOut[i]);
+            uint256 _tokenAmountOut = tokenAmountsOut[i];
+            if (_tokenAmountOut < minTokenAmountsOut[i]) {
+                revert SlippageOut(_tokenAmountOut, minTokenAmountsOut[i]);
             }
-            _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);
-            reserves[i] = reserves[i] - tokenAmountsOut[i];
+            _tokens[i].safeTransfer(recipient, _tokenAmountOut);
+            reserves[i] = reserves[i] - _tokenAmountOut;
         }

         _setReserves(_tokens, reserves);
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L548-L560

*Gas Savings for `Well.removeLiquidityImbalanced`, obtained via protocol's tests: Med 70 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  89489   |  97989   |  72642  |    5     |
| After  |  89419   |  97919   |  72586  |    5     |


### Can cache `tokenAmountsOut[i]` since it is accessed mutliple times
```solidity
File: src/Well.sol
548:    function removeLiquidityImbalanced(
549:        uint256 maxLpAmountIn,
550:        uint256[] calldata tokenAmountsOut,
551:        address recipient,
552:        uint256 deadline
553:    ) external nonReentrant expire(deadline) returns (uint256 lpAmountIn) {
...
557:        for (uint256 i; i < _tokens.length; ++i) {
558:            _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);
559:            reserves[i] = reserves[i] - tokenAmountsOut[i];
560:        }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..5a303b4 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -555,8 +555,9 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
         uint256[] memory reserves = _updatePumps(_tokens.length);

         for (uint256 i; i < _tokens.length; ++i) {
-            _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);
-            reserves[i] = reserves[i] - tokenAmountsOut[i];
+            uint256 _tokenAmountOut = tokenAmountsOut[i];
+            _tokens[i].safeTransfer(recipient, _tokenAmountOut);
+            reserves[i] = reserves[i] - _tokenAmountOut;
         }

         lpAmountIn = totalSupply() - _calcLpTokenSupply(wellFunction(), reserves);
```

## Refactor function to avoid potentially iterating through array twice
The instances below point to occurances in which a memory array is potentially iterated through twice. The second iteration is done in an internal function and thus we can combine some of the logic of the internal function with the for loop in the parent function in order to potentially avoid iterating through the array more than once.

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L352-L366

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L759-L764

*Gas Savings for `Well.shift`, obtained via protocol's tests: Max 157 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  38386   |  59296   |  36740  |    6     |
| After  |  38339   |  59139   |  36655  |    6     |

*Note: This optimization will save the most gas in the event where the `tokenOut` is in a higher position in the array, thus resulting in the function below potentially iterating through a majority of the array twice. Since the greatest gas savings will be seen in the specified case above, the `Max` column is used to benchmark this particular instance.*

```solidity
File: src/Well.sol
352:    function shift(
353:        IERC20 tokenOut,
354:        uint256 minAmountOut,
355:        address recipient
356:    ) external nonReentrant returns (uint256 amountOut) {
...
363:        for (uint256 i; i < _tokens.length; ++i) {
364:            reserves[i] = _tokens[i].balanceOf(address(this));
365:        }
366:        uint256 j = _getJ(_tokens, tokenOut); // @audit: `_tokens` is iterated through again (# of iterations vary).

759:    function _getJ(IERC20[] memory _tokens, IERC20 jToken) internal pure returns (uint256 j) {
760:        for (j; j < _tokens.length; ++j) {
761:            if (jToken == _tokens[j]) {
762:                return j;
763:            }
764:        }
```
```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..6427173 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -360,10 +360,16 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
         // Use the balances of the pool instead of the stored reserves.
         // If there is a change in token balances relative to the currently
         // stored reserves, the extra tokens can be shifted into `tokenOut`.
+        uint256 j;
+        bool completed;
         for (uint256 i; i < _tokens.length; ++i) {
             reserves[i] = _tokens[i].balanceOf(address(this));
+            if (tokenOut == _tokens[i]) {
+                j = i;
+                completed = true;
+            }
         }
-        uint256 j = _getJ(_tokens, tokenOut);
+        if (!completed) revert InvalidTokens();
         amountOut = reserves[j] - _calcReserve(wellFunction(), reserves, j, totalSupply());

         if (amountOut >= minAmountOut) {
```

*Note: This optimization only saves gas in the case that the pump has never been used before. The `for loop` in `update` and the `for loop` in `_init` both iterate through the array and perform the same check: `if (reserves[i] == 0) return;`, with the loop in `_init` additionally extending another array. We can combine these for loops to avoid interating through the `reserves` array twice in the case where the pump has never been used before. This optimization would benefit the following function: `MultiFlowPump.update`*

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L84-L88

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L146-L155

```solidity
File: src/pumps/MultiFlowPump.sol
84:      for (uint256 i; i < numberOfReserves; ++i) { // @audit: `reserves` array is iterated through
85:          // If a reserve is 0, then the pump cannot be initialized.
86:          if (reserves[i] == 0) return;
87:       }
88:      _init(slot, uint40(block.timestamp), reserves); // @audit: if this section is reached, all items in `reserves` != 0

146:    function _init(bytes32 slot, uint40 lastTimestamp, uint256[] memory reserves) internal {
147:        uint256 numberOfReserves = reserves.length;
148:        bytes16[] memory byteReserves = new bytes16[](numberOfReserves);
149:
150:        // Skip {_capReserve} since we have no prior reference
151:
152:        for (uint256 i; i < numberOfReserves; ++i) { // @audit: `reserves` array is iterated through again
153:            if (reserves[i] == 0) return; // @audit: this line is redundant due to first loop in parent function
154:            byteReserves[i] = reserves[i].fromUIntToLog2();
155:        }
```
```diff
diff --git a/src/pumps/MultiFlowPump.sol b/src/pumps/MultiFlowPump.sol
index e2aab5d..a06e215 100644
--- a/src/pumps/MultiFlowPump.sol
+++ b/src/pumps/MultiFlowPump.sol
@@ -81,11 +81,15 @@ contract MultiFlowPump is IPump, IMultiFlowPumpErrors, IInstantaneousPump, ICumu

         // If the last timestamp is 0, then the pump has never been used before.
         if (pumpState.lastTimestamp == 0) {
+            bytes16[] memory byteReserves = new bytes16[](numberOfReserves);
+
+            // Skip {_capReserve} since we have no prior reference
+
             for (uint256 i; i < numberOfReserves; ++i) {
-                // If a reserve is 0, then the pump cannot be initialized.
                 if (reserves[i] == 0) return;
+                byteReserves[i] = reserves[i].fromUIntToLog2();
             }
-            _init(slot, uint40(block.timestamp), reserves);
+            _init(slot, uint40(block.timestamp), byteReserves);
             return;
         }

@@ -143,17 +147,7 @@ contract MultiFlowPump is IPump, IMultiFlowPumpErrors, IInstantaneousPump, ICumu
      * @dev On first update for a particular Well, initialize oracle with
      * reserves data.
      */
-    function _init(bytes32 slot, uint40 lastTimestamp, uint256[] memory reserves) internal {
-        uint256 numberOfReserves = reserves.length;
-        bytes16[] memory byteReserves = new bytes16[](numberOfReserves);
-
-        // Skip {_capReserve} since we have no prior reference
-
-        for (uint256 i; i < numberOfReserves; ++i) {
-            if (reserves[i] == 0) return;
-            byteReserves[i] = reserves[i].fromUIntToLog2();
-        }
-
+    function _init(bytes32 slot, uint40 lastTimestamp, bytes16[] memory byteReserves) internal {
         // Write: Last Timestamp & Last Reserves
         slot.storeLastReserves(lastTimestamp, byteReserves);
```

## Cache External Call
External calls are expensive as they use the `CALL/STATICCALL` opcode (~100 gas for warm addresses). If an external call is performed more than once, it should be cached in order to avoid incurring the cost twice.

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L67-L76

*Note: The instance below refers to a library function, and is not explicitly tested in the test suite. This optimization assumes that this library/function will be used within state-mutating functions within contracts that integrate with Basin. This instance is not included in the final Diffs, but is included here for completeness.*

### Cache `LibContractInfo.getSymbol(address(_tokens[i]))` to save 1 External Call. **`LibContractInfo.getSymbol` performs one `STATICCALL`.**
```solidity
File: src/libraries/LibWellConstructor.sol
67:    function encodeWellInitFunctionCall(
68:        IERC20[] memory _tokens,
69:        Call memory _wellFunction
70:    ) public view returns (bytes memory initFunctionCall) {
71:        string memory name = LibContractInfo.getSymbol(address(_tokens[0]));
72:        string memory symbol = name;
73:        for (uint256 i = 1; i < _tokens.length; ++i) {
74:            name = string.concat(name, ":", LibContractInfo.getSymbol(address(_tokens[i]))); // @audit: 1st external call to `_tokens[i]`
75:            symbol = string.concat(symbol, LibContractInfo.getSymbol(address(_tokens[i]))); // @audit: 2nd external call to `_tokens[i]`
76:        }
```
```diff
diff --git a/src/libraries/LibWellConstructor.sol b/src/libraries/LibWellConstructor.sol
index 507f2d9..457a85f 100644
--- a/src/libraries/LibWellConstructor.sol
+++ b/src/libraries/LibWellConstructor.sol
@@ -70,9 +70,11 @@ library LibWellConstructor {
     ) public view returns (bytes memory initFunctionCall) {
         string memory name = LibContractInfo.getSymbol(address(_tokens[0]));
         string memory symbol = name;
+        string memory temp;
         for (uint256 i = 1; i < _tokens.length; ++i) {
-            name = string.concat(name, ":", LibContractInfo.getSymbol(address(_tokens[i])));
-            symbol = string.concat(symbol, LibContractInfo.getSymbol(address(_tokens[i])));
+            temp = LibContractInfo.getSymbol(address(_tokens[i]));
+            name = string.concat(name, ":", temp);
+            symbol = string.concat(symbol, temp);
         }
         name = string.concat(name, " ", LibContractInfo.getName(_wellFunction.target), " Well");
         symbol = string.concat(symbol, LibContractInfo.getSymbol(_wellFunction.target), "w");
```

## Sort array offchain to check duplicates in O(n) instead of O(n^2)
Instead of using two for loops to check for duplicates, which runs in O(n^2) time and is expensive, the `tokens` array can be sorted offchain which allows us to to check duplicates by simply ensuring the the current id is larger than the previous one (O(n) time). Seeing as this `Well` is deployed using dynamic immutable storage that needs to be pre-encoded, it is within reason that the tokens array can be sorted before the encoding.

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L35-L42

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L47-L50:

```solidity
File: src/Well.sol
35:        IERC20[] memory _tokens = tokens();
36:        for (uint256 i; i < _tokens.length - 1; ++i) {
37:            for (uint256 j = i + 1; j < _tokens.length; ++j) {
38:                if (_tokens[i] == _tokens[j]) {
39:                    revert DuplicateTokens(_tokens[i]);
40:                }
41:            }
42:        }

47:    /// This Well uses a dynamic immutable storage layout. Immutable storage is
48:    /// used for gas-efficient reads during Well operation. The Well must be
49:    /// created by cloning with a pre-encoded byte string containing immutable // @audit: immutable data needs to be `pre-encoded`
50:    /// data.
```

## Gas Report output with all optimizations applied

```js
| mocks/functions/MockEmptyFunction.sol:MockEmptyFunction contract |                 |      |        |      |         |
|------------------------------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                                                  | Deployment Size |      |        |      |         |
| 202045                                                           | 1041            |      |        |      |         |
| Function Name                                                    | min             | avg  | median | max  | # calls |
| calcLpTokenSupply                                                | 1052            | 1052 | 1052   | 1052 | 1       |
| calcReserve                                                      | 1085            | 1085 | 1085   | 1085 | 1       |
| name                                                             | 328             | 328  | 328    | 328  | 1       |
| symbol                                                           | 349             | 349  | 349    | 349  | 1       |


| mocks/functions/MockFunctionBad.sol:MockFunctionBad contract |                 |      |        |      |         |
|--------------------------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                                              | Deployment Size |      |        |      |         |
| 228469                                                       | 1173            |      |        |      |         |
| Function Name                                                | min             | avg  | median | max  | # calls |
| calcReserve                                                  | 1085            | 1085 | 1085   | 1085 | 1       |
| name                                                         | 328             | 328  | 328    | 328  | 1       |
| symbol                                                       | 349             | 349  | 349    | 349  | 1       |


| mocks/pumps/MockFailPump.sol:MockFailPump contract |                 |     |        |     |         |
|----------------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                                    | Deployment Size |     |        |     |         |
| 60111                                              | 332             |     |        |     |         |
| Function Name                                      | min             | avg | median | max | # calls |
| update                                             | 541             | 541 | 541    | 541 | 4       |


| mocks/pumps/MockPump.sol:MockPump contract |                 |      |        |       |         |
|--------------------------------------------|-----------------|------|--------|-------|---------|
| Deployment Cost                            | Deployment Size |      |        |       |         |
| 254494                                     | 1303            |      |        |       |         |
| Function Name                              | min             | avg  | median | max   | # calls |
| update                                     | 1207            | 4600 | 3207   | 23121 | 180     |


| mocks/tokens/MockToken.sol:MockToken contract |                 |       |        |       |         |
|-----------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                               | Deployment Size |       |        |       |         |
| 1199989                                       | 7259            |       |        |       |         |
| Function Name                                 | min             | avg   | median | max   | # calls |
| approve                                       | 24651           | 24651 | 24651  | 24651 | 557     |
| balanceOf                                     | 563             | 939   | 563    | 2563  | 595     |
| burnFrom                                      | 5266            | 10066 | 10066  | 14866 | 4       |
| mint                                          | 10543           | 31889 | 24843  | 46743 | 556     |
| name                                          | 3241            | 3241  | 3241   | 3241  | 1       |
| symbol                                        | 1262            | 1302  | 1262   | 3262  | 299     |
| transfer                                      | 2476            | 14913 | 12694  | 29794 | 41      |
| transferFrom                                  | 2860            | 16564 | 20380  | 20380 | 252     |


| mocks/tokens/MockTokenFeeOnTransfer.sol:MockTokenFeeOnTransfer contract |                 |       |        |       |         |
|-------------------------------------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                                                         | Deployment Size |       |        |       |         |
| 1252648                                                                 | 7516            |       |        |       |         |
| Function Name                                                           | min             | avg   | median | max   | # calls |
| approve                                                                 | 24675           | 24675 | 24675  | 24675 | 63      |
| balanceOf                                                               | 586             | 1080  | 586    | 2586  | 93      |
| fee                                                                     | 2383            | 2383  | 2383   | 2383  | 3       |
| mint                                                                    | 24821           | 32121 | 24821  | 46721 | 63      |
| setFee                                                                  | 20301           | 20301 | 20301  | 20301 | 18      |
| symbol                                                                  | 1306            | 1306  | 1306   | 1306  | 30      |
| transfer                                                                | 16445           | 18578 | 19645  | 19645 | 3       |
| transferFrom                                                            | 16712           | 21270 | 22712  | 22712 | 34      |


| mocks/wells/MockInitFailWell.sol:MockInitFailWell contract |                 |     |        |     |         |
|------------------------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                                            | Deployment Size |     |        |     |         |
| 39293                                                      | 226             |     |        |     |         |
| Function Name                                              | min             | avg | median | max | # calls |
| initMessage                                                | 228             | 228 | 228    | 228 | 1       |
| initNoMessage                                              | 107             | 107 | 107    | 107 | 1       |


| mocks/wells/MockReserveWell.sol:MockReserveWell contract |                 |       |        |       |         |
|----------------------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                                          | Deployment Size |       |        |       |         |
| 277882                                                   | 1434            |       |        |       |         |
| Function Name                                            | min             | avg   | median | max   | # calls |
| getReserves                                              | 1169            | 2835  | 1169   | 7169  | 36      |
| setReserves                                              | 1516            | 1516  | 1516   | 1516  | 6       |
| update                                                   | 45896           | 78071 | 78156  | 99949 | 30028   |


| mocks/wells/MockStaticWell.sol:MockStaticWell contract |                 |       |        |       |         |
|--------------------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                                        | Deployment Size |       |        |       |         |
| 560963                                                 | 4891            |       |        |       |         |
| Function Name                                          | min             | avg   | median | max   | # calls |
| aquifer                                                | 185             | 185   | 185    | 185   | 10      |
| immutableDataFromClone                                 | 275             | 275   | 275    | 275   | 2       |
| init                                                   | 69982           | 69982 | 69982  | 69982 | 5       |
| name                                                   | 1232            | 1232  | 1232   | 1232  | 4       |
| pumps                                                  | 1409            | 1409  | 1409   | 1409  | 9       |
| symbol                                                 | 1232            | 1232  | 1232   | 1232  | 4       |
| tokens                                                 | 886             | 886   | 886    | 886   | 9       |
| wellData                                               | 5922            | 5922  | 5922   | 5922  | 9       |
| wellFunction                                           | 1038            | 1038  | 1038   | 1038  | 9       |


| src/Aquifer.sol:Aquifer contract |                 |        |        |        |         |
|----------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                  | Deployment Size |        |        |        |         |
| 748666                           | 3666            |        |        |        |         |
| Function Name                    | min             | avg    | median | max    | # calls |
| boreWell                         | 50047           | 337170 | 359143 | 399561 | 124     |
| wellImplementation               | 527             | 527    | 527    | 527    | 5       |


| src/Well.sol:Well contract     |                 |        |        |        |         |
|--------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                | Deployment Size |        |        |        |         |
| 4009572                        | 20057           |        |        |        |         |
| Function Name                  | min             | avg    | median | max    | # calls |
| addLiquidity                   | 6115            | 129283 | 131499 | 198579 | 140     |
| addLiquidityFeeOnTransfer      | 6116            | 61536  | 70657  | 109097 | 13      |
| aquifer                        | 296             | 296    | 296    | 296    | 116     |
| balanceOf                      | 598             | 1598   | 1598   | 2598   | 136     |
| decimals                       | 246             | 246    | 246    | 246    | 1       |
| firstPump                      | 3031            | 3031   | 3031   | 3031   | 1       |
| getAddLiquidityOut             | 7097            | 12449  | 13597  | 13597  | 17      |
| getRemoveLiquidityImbalancedIn | 6777            | 11177  | 11277  | 13277  | 5       |
| getRemoveLiquidityOneTokenOut  | 13309           | 13309  | 13309  | 13309  | 2       |
| getRemoveLiquidityOut          | 7148            | 10398  | 10398  | 13648  | 2       |
| getReserves                    | 1426            | 2102   | 1426   | 9523   | 80      |
| getShiftOut                    | 11080           | 11208  | 11273  | 11273  | 3       |
| getSwapIn                      | 4753            | 9312   | 9312   | 13872  | 2       |
| getSwapOut                     | 8641            | 13150  | 13900  | 13910  | 7       |
| init                           | 121809          | 200826 | 210579 | 212531 | 116     |
| name                           | 7593            | 7593   | 7593   | 7593   | 1       |
| numberOfPumps                  | 683             | 683    | 683    | 683    | 1       |
| pumps                          | 878             | 3779   | 3619   | 5433   | 117     |
| removeLiquidity                | 5797            | 53056  | 34937  | 119211 | 8       |
| removeLiquidityImbalanced      | 5774            | 72504  | 89303  | 97803  | 5       |
| removeLiquidityOneToken        | 5671            | 42322  | 41310  | 80999  | 4       |
| shift                          | 13877           | 36546  | 38220  | 59020  | 6       |
| skim                           | 52970           | 52970  | 52970  | 52970  | 1       |
| swapFrom                       | 5793            | 87997  | 100380 | 150596 | 35      |
| swapFromFeeOnTransfer          | 5771            | 45314  | 49868  | 66924  | 7       |
| swapTo                         | 5771            | 41283  | 28342  | 89012  | 7       |
| symbol                         | 3329            | 3329   | 3329   | 3329   | 1       |
| sync                           | 15891           | 15891  | 15891  | 15891  | 5       |
| tokens                         | 1544            | 1567   | 1544   | 1762   | 120     |
| totalSupply                    | 0               | 658    | 405    | 2405   | 164     |
| well                           | 7380            | 7380   | 7380   | 7380   | 1       |
| wellData                       | 524             | 524    | 524    | 524    | 116     |
| wellFunction                   | 2049            | 2059   | 2049   | 2758   | 137     |
| wellFunctionAddress            | 542             | 542    | 542    | 542    | 1       |


| src/functions/ConstantProduct.sol:ConstantProduct contract |                 |      |        |      |         |
|------------------------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                                            | Deployment Size |      |        |      |         |
| 621657                                                     | 3137            |      |        |      |         |
| Function Name                                              | min             | avg  | median | max  | # calls |
| calcLpTokenSupply                                          | 1965            | 2301 | 1965   | 3275 | 5       |
| calcReserveAtRatioLiquidity                                | 1896            | 1896 | 1896   | 1896 | 10      |
| calcReserveAtRatioSwap                                     | 2870            | 2877 | 2870   | 2918 | 10      |
| name                                                       | 465             | 465  | 465    | 465  | 1       |
| symbol                                                     | 525             | 525  | 525    | 525  | 1       |


| src/functions/ConstantProduct2.sol:ConstantProduct2 contract |                 |      |        |      |         |
|--------------------------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                                              | Deployment Size |      |        |      |         |
| 536368                                                       | 2711            |      |        |      |         |
| Function Name                                                | min             | avg  | median | max  | # calls |
| calcLPTokenUnderlying                                        | 1807            | 1807 | 1807   | 1807 | 10      |
| calcLpTokenSupply                                            | 780             | 1423 | 1429   | 1477 | 209     |
| calcReserve                                                  | 1787            | 1791 | 1787   | 1797 | 71      |
| calcReserveAtRatioLiquidity                                  | 1434            | 1439 | 1439   | 1444 | 10      |
| calcReserveAtRatioSwap                                       | 1906            | 1925 | 1916   | 1988 | 10      |
| name                                                         | 465             | 465  | 465    | 465  | 115     |
| symbol                                                       | 525             | 525  | 525    | 525  | 115     |


| src/pumps/MultiFlowPump.sol:MultiFlowPump contract |                 |       |        |       |         |
|----------------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                                    | Deployment Size |       |        |       |         |
| 3307470                                            | 18692           |       |        |       |         |
| Function Name                                      | min             | avg   | median | max   | # calls |
| readCumulativeReserves                             | 13653           | 49134 | 47838  | 59888 | 17      |
| readInstantaneousReserves                          | 13652           | 53582 | 57706  | 65490 | 7       |
| readLastCumulativeReserves                         | 1683            | 1855  | 1683   | 2544  | 5       |
| readLastInstantaneousReserves                      | 2521            | 10601 | 12058  | 14212 | 5       |
| readLastReserves                                   | 2662            | 10123 | 11365  | 11421 | 7       |
| readTwaReserves                                    | 13966           | 55576 | 60773  | 65139 | 11      |
| update                                             | 3294            | 75201 | 75352  | 97147 | 30077   |


| test/helpers/Users.sol:Users contract |                 |      |        |      |         |
|---------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                       | Deployment Size |      |        |      |         |
| 768927                                | 3630            |      |        |      |         |
| Function Name                         | min             | avg  | median | max  | # calls |
| createUsers                           | 4132            | 6186 | 6632   | 6632 | 129     |
| getNextUserAddress                    | 1025            | 2109 | 1025   | 5825 | 262     |
```