
### Gas Optimization Issues
|Title|Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
|[G-1] Use assembly to emit events | [Use assembly to emit events](#use-assembly-to-emit-events) | 8 | 304 |
|[G-2] Check Arguments Early | [Check Arguments Early](#check-arguments-early) | 3 | - |
|[G-3] Division operations between unsigned could be unchecked | [Division operations between unsigned could be unchecked](#division-operations-between-unsigned-could-be-unchecked) | 16 | 1360 |
|[G-4] Empty Block | [Empty Block](#empty-block) | 3 | - |
|[G-5] Cache External Calls in Loops | [Cache External Calls in Loops](#cache-external-calls-in-loops) | 5 | 500 |
|[G-6] Use a More Recent Version of Solidity | [Use a More Recent Version of Solidity](#use-a-more-recent-version-of-solidity) | 27 | - |
|[G-7] Greater or Equal Comparison Costs Less Gas than Greater Comparison | [Greater or Equal Comparison Costs Less Gas than Greater Comparison](#greater-or-equal-comparison-costs-less-gas-than-greater-comparison) | 3 | 9 |
|[G-8] Consider Merge Sequential For-Loops | [Consider Merge Sequential For-Loops](#consider-merge-sequential-for-loops) | 2 | - |
|[G-9] Consider activating via-ir for deploying | [Consider activating via-ir for deploying](#consider-activating-via-ir-for-deploying) | 1 | - |
|[G-10] Safe Subtraction Should Be Unchecked | [Safe Subtraction Should Be Unchecked](#safe-subtraction-should-be-unchecked) | 31 | 2635 |
|[G-11] Unnecessary Casting of Variables | [Unnecessary Casting of Variables](#unnecessary-casting-of-variables) | 3 | - |
|[G-12] Avoid Unnecessary Computation Inside Loops | [Avoid Unnecessary Computation Inside Loops](#avoid-unnecessary-computation-inside-loops) | 2 | - |
|[G-13] Redundant Contract Existence Check in Consecutive External Calls | [Redundant Contract Existence Check in Consecutive External Calls](#redundant-contract-existence-check-in-consecutive-external-calls) | 2 | 200 |
|[G-14] Unused Named Return Variables | [Unused Named Return Variables](#unused-named-return-variables) | 1 | - |

Total: 268 instances over 27 issues

#

## Use assembly to emit events
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 304

### Description
This detector checks for instances where a contract uses assembly code to emit events. While it's technically possible to emit events using inline assembly in Solidity, it's generally discouraged due to readability concerns and potential for errors. Events should usually be emitted using higher-level Solidity constructs.

<details>

<summary>
There are 8 instances of this issue:

</summary>

###
- File: src/Well.sol
```
 
Line: 238          emit Swap(fromToken, toToken, amountIn, amountOut, recipient)
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L238](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L238)


- File: src/Well.sol
```
 
Line: 305          emit Swap(fromToken, toToken, amountIn, amountOut, recipient)
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L305](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L305)


- File: src/Well.sol
```
 
Line: 373          emit Shift(reserves, tokenOut, amountOut, recipient)
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L373](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L373)


- File: src/Well.sol
```
 
Line: 443          emit AddLiquidity(tokenAmountsIn, lpAmountOut, recipient)
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L443](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L443)


- File: src/Well.sol
```
 
Line: 482          emit RemoveLiquidity(lpAmountIn, tokenAmountsOut, recipient)
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L482](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L482)


- File: src/Well.sol
```
 
Line: 516          emit RemoveLiquidityOneToken(lpAmountIn, tokenOut, tokenAmountOut, recipient)
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L516](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L516)


- File: src/Well.sol
```
 
Line: 569          emit RemoveLiquidity(lpAmountIn, tokenAmountsOut, recipient)
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L569](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L569)


- File: src/Well.sol
```
 
Line: 597          emit Sync(reserves)
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L597](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L597)


</details>

# 

## Empty Block
- Severity: Gas Optimization
- Confidence: High

### Description
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: src/Well.sol
```
 
Line: 656          try IPump(_pump.target).update(reserves, _pump.data) {}

            catch {

                // ignore reversion. If an external shutoff mechanism is added to a Pump, it could be called here.

            }
```
  contains an empty block.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L656-L659](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L656-L659)


- File: src/Well.sol
```
 
Line: 664          try IPump(_pumps[i].target).update(reserves, _pumps[i].data) {}

                catch {

                    // ignore reversion. If an external shutoff mechanism is added to a Pump, it could be called here.

                }
```
  contains an empty block.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L664-L667](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L664-L667)


- File: src/Well.sol
```
 
Line: 653          if (numberOfPumps() == 1) {

            Call memory _pump = firstPump();

            // Don't revert if the update call fails.

            try IPump(_pump.target).update(reserves, _pump.data) {}

            catch {

                // ignore reversion. If an external shutoff mechanism is added to a Pump, it could be called here.

            }

        } else {

            Call[] memory _pumps = pumps();

            for (uint256 i; i < _pumps.length; ++i) {

                // Don't revert if the update call fails.

                try IPump(_pumps[i].target).update(reserves, _pumps[i].data) {}

                catch {

                    // ignore reversion. If an external shutoff mechanism is added to a Pump, it could be called here.

                }

            }

        }
```
  contains an empty block.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L653-L669](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L653-L669)


</details>

# 

## Check Arguments Early
- Severity: Gas Optimization
- Confidence: High

### Description
Checks that require() or revert() statements that check input arguments are at the top of the function. Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a gas load in a function that may ultimately revert in the unhappy case.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 221          require(uint128(x) < 0x80000000000000000000000000000000);
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L221](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L221)


- File: src/libraries/LibBytes.sol
```
 
Line: 63          require(reserves[reserves.length - 1] <= type(uint128).max, "ByteStorage: too large")
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L63](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L63)


- File: src/libraries/LibMath.sol
```
 
Line: 159          require(denominator > 0);

```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibMath.sol#L159](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibMath.sol#L159)


</details>

# 

## Division operations between unsigned could be unchecked
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 1360

### Description
Division operations on unsigned integers should be unchecked to save gas since they cannot overflow or underflow. Because unsigned integers cannot have negative values, execution of division operations outside `unchecked` blocks adds nothing but overhead. Saves about 85 gas.

<details>

<summary>
There are 16 instances of this issue:

</summary>

###
- File: src/functions/ConstantProduct.sol
```
 
Line: 40          erve = uint256((lpTokenSupply / n) ** n);


```
 Should be unchecked - lpTokenSupply / n.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct.sol#L40](https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct.sol#L40)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 918          Signifier = xSignifier / ySignifier;
```
 Should be unchecked - xSignifier / ySignifier.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L918](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L918)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1035           = (r + xSignifier / r) >> 1;
```
 Should be unchecked - xSignifier / r.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1035](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1035)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1036           = (r + xSignifier / r) >> 1;
```
 Should be unchecked - xSignifier / r.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1036](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1036)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1037           = (r + xSignifier / r) >> 1;
```
 Should be unchecked - xSignifier / r.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1037](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1037)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1038           = (r + xSignifier / r) >> 1;
```
 Should be unchecked - xSignifier / r.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1038](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1038)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1039           = (r + xSignifier / r) >> 1;
```
 Should be unchecked - xSignifier / r.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1039](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1039)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1040           = (r + xSignifier / r) >> 1;
```
 Should be unchecked - xSignifier / r.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1040](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1040)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1041           = (r + xSignifier / r) >> 1;
```
 Should be unchecked - xSignifier / r.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1041](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1041)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1042          int256 r1 = xSignifier / r;
```
 Should be unchecked - xSignifier / r.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1042](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1042)





- File: src/libraries/LibMath.sol
```
 
Line: 52          xNew = x - (x - a0 / t0) / n
```
 Should be unchecked - a0 / t0.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibMath.sol#L52](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibMath.sol#L52)


- File: src/libraries/LibMath.sol
```
 
Line: 54          xNew = x + (a0 / t0 - x) / n
```
 Should be unchecked - a0 / t0.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibMath.sol#L54](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibMath.sol#L54)




- File: src/libraries/LibMath.sol
```
 
Line: 198          od0 := div(prod0, twos)


```
 Should be unchecked - prod0 / twos.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibMath.sol#L198](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibMath.sol#L198)


- File: src/pumps/MultiFlowPump.sol
```
 
Line: 113          blocksPassed = (deltaTimestamp / BLOCK_TIME).fromUInt()
```
 Should be unchecked - deltaTimestamp / BLOCK_TIME.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L113](https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L113)


- File: src/pumps/MultiFlowPump.sol
```
 
Line: 252          tes16 blocksPassed = (deltaTimestamp / BLOCK_TIME).fromUInt();

```
 Should be unchecked - deltaTimestamp / BLOCK_TIME.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L252](https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L252)


- File: src/pumps/MultiFlowPump.sol
```
 
Line: 299          tes16 blocksPassed = (deltaTimestamp / BLOCK_TIME).fromUInt();

```
 Should be unchecked - deltaTimestamp / BLOCK_TIME.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L299](https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L299)


</details>

# 



## Avoid Unnecessary Computation Inside Loops
- Severity: Gas Optimization
- Confidence: High

### Description
This detector identifies instances of computations being performed inside loops that could be performed outside the loop to save gas.

Unnecessary computation inside loops:

Any computation performed inside a loop that doesn't change with each iteration can be moved outside the loop to save gas.This detector identifies instances where the following unnecessary computations are performed inside loops:

- 1 Local variables are read inside loops but never modified within the same loop, indicating they could be cached outside the loop.

- 2 Computations involving constant expressions (literals) which result in the same output in every loop iteration.

By addressing these issues, gas usage can be optimized.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: src/Well.sol
```
 
Line: 36          i < _tokens.length - 1
```
The following computations can be cached outside of loops for gas optimization `_tokens.length - 1`<br>.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L36](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L36)


- File: src/libraries/LibMath.sol
```
 
Line: 50          uint256 t0 = x ** (n - 1)
```
The following computations can be cached outside of loops for gas optimization `n - 1`<br>.
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibMath.sol#L50](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibMath.sol#L50)


</details>

# 


## Cache External Calls in Loops
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 500

### Description
This detector identifies instances of repeated external calls within loops. By moving the first external call outside of the loop and using low-level calls inside the loop, it is possible to save gas by avoiding repeated EXTCODESIZE opcodes, as there is no need to check again if the contract exists. 

Note: This detector only triggers if the function call does not return any value.

Prior to 0.8.10, the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent Solidity versions the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low-level calls never check for contract existence. 

<details>

<summary>
There are 5 instances of this issue:

</summary>

###
- File: src/Well.sol
```
 
Line: 431          _tokens[i].safeTransferFrom(msg.sender, address(this), tokenAmountsIn[i])
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L431](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L431)


- File: src/Well.sol
```
 
Line: 477          _tokens[i].safeTransfer(recipient, tokenAmountsOut[i])
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L477](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L477)


- File: src/Well.sol
```
 
Line: 558          _tokens[i].safeTransfer(recipient, tokenAmountsOut[i])
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L558](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L558)


- File: src/Well.sol
```
 
Line: 610          _tokens[i].safeTransfer(recipient, skimAmounts[i])
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L610](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L610)


- File: src/Well.sol
```
 
Line: 664          try IPump(_pumps[i].target).update(reserves, _pumps[i].data) {}

                catch {

                    // ignore reversion. If an external shutoff mechanism is added to a Pump, it could be called here.

                }
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L664-L667](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L664-L667)


</details>

# 
## Greater or Equal Comparison Costs Less Gas than Greater Comparison
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 9

### Description
In Solidity, the >= operator costs less gas than the > operator. This is because the Solidity compiler uses the LT opcode for >= comparisons, which requires less gas than the combination of GT and ISZERO opcodes used for > comparisons. Therefore, replacing > with >= when possible can reduce the gas cost of your contract.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 57          int256 result = uint256(x > 0 ? x : -x);
```
 using GREATER_EQUAL/LESS_EQUAL in this case is identical therefore should be use.

[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L57](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L57)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 246          int256 result = uint256(x > 0 ? x : -x);
```
 using GREATER_EQUAL/LESS_EQUAL in this case is identical therefore should be use.

[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L246](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L246)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 303          int256 result = uint128(x > 0 ? x : -x);
```
 using GREATER_EQUAL/LESS_EQUAL in this case is identical therefore should be use.

[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L303](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L303)


</details>

#

## Consider Merge Sequential For-Loops
- Severity: Gas Optimization
- Confidence: High

### Description
Detects instances where multiple for-loops appear sequentially in the same function. This may indicate an opportunity for loop fusion, a code optimization technique that merges multiple loops into a single one.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- Consider merging those following loops to one loop if appropriate :File: src/Well.sol
```
 
Line: 423          i < _tokens.length
```
File: src/Well.sol
```
 
Line: 429          i < _tokens.length
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L423](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L423)


- Consider merging those following loops to one loop if appropriate :File: src/pumps/MultiFlowPump.sol
```
 
Line: 84          i < numberOfReserves
```
File: src/pumps/MultiFlowPump.sol
```
 
Line: 116          i < numberOfReserves
```

[https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L84](https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L84)


</details>

# 

## Consider activating via-ir for deploying
- Severity: Gas Optimization
- Confidence: Medium

### Description
The IR-based code generator was developed to make code generation more transparent and auditable, while also enabling powerful optimization passes that can be applied across functions. 

It is possible to activate the IR-based code generator through the command line by using the flag --via-ir or by including the option {'viaIR': true}. 

Keep in mind that compiling with this option may take longer. However, you can simply test it before deploying your code. If you find that it provides better performance, you can add the --via-ir flag to your deploy command.

#


## Redundant Contract Existence Check in Consecutive External Calls
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 200

### Description
This detector identifies instances where there are consecutive external calls to the same contract, where the subsequent calls could use a low-level call to save gas.

Note: This detector only triggers if the function call does not return any value. 

Prior to 0.8.10, the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent Solidity versions the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low-level calls never check for contract existence. 

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: src/Well.sol
```
 
Line: 304          toToken.safeTransfer(recipient, amountOut)
```
This call could be replaced with a low-level call because the contract `SafeERC20` has already been checked in <br>`Line: 303    fromToken.safeTransferFrom(msg.sender, address(this), amountIn)`<br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L304](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L304)


- File: src/pumps/MultiFlowPump.sol
```
 
Line: 133          slot.storeBytes16(pumpState.emaReserves)
```
This call could be replaced with a low-level call because the contract `LibBytes16` has already been checked in <br>`Line: 129    slot.storeBytes16(pumpState.cumulativeReserves)`<br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L133](https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L133)


</details>

# 

## Unused Named Return Variables
- Severity: Gas Optimization
- Confidence: High

### Description
Named return variables allow for clear and explicit naming of values to be returned from a function. However, when these variables are unused, it can lead to confusion and make the code less maintainable.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: src/Aquifer.sol
```
 
Line: 86          function wellImplementation(address well) external view returns (address implementation) 
```
there is not use of this variables:
@ **implementation**

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L86-L88](https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L86-L88)


</details>

# 

## Safe Subtraction Should Be Unchecked
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 2635

### Description
This detector identifies subtraction operations where the subtraction could safely be unchecked. This occurs when a prior if-statement or require() function ensures that the first operand (minuend) is greater than or equal to the second operand (subtrahend). In these cases, an unchecked subtraction would not result in an underflow error, and can save gas by skipping the redundant check.

<details>

<summary>
There are 31 instances of this issue:

</summary>

###
- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 61          esult >>= msb - 112;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 61          sb > 112)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L61](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L61)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 57          int256 result = uint256(x > 0 ? x : -x);
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 53           == 0)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L57](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L57)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 60          esult <<= 112 - msb;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 60          sb < 112)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L60](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L60)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 144          esultSignifier = xExponent - 0x3FFF;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 142          Exponent >= 0x3FFF)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L144](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L144)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 115          esult <<= 112 - msb;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 115          sb < 112)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L115](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L115)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 116          esult >>= msb - 112;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 116          sb > 112)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L116](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L116)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 196          esult <<= 112 - msb;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 196          sb < 112)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L196](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L196)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 197          esult >>= msb - 112;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 197          sb > 112)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L197](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L197)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 249          esult <<= 112 - msb;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 249          sb < 112)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L249](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L249)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 250          esult >>= msb - 112;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 250          sb > 112)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L250](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L250)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 246          int256 result = uint256(x > 0 ? x : -x);
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 242           == 0)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L246](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L246)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 307          esult >>= msb - 112;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 307          sb > 112)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L307](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L307)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 303          int256 result = uint128(x > 0 ? x : -x);
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 299           == 0)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L303](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L303)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 306          esult <<= 112 - msb;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 306          sb < 112)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L306](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L306)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 712          int256 shift = 112 - msb;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 711          sb < 112)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L712](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L712)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 715          Exponent -= shift;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 713          Exponent > shift)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L715](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L715)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 696          Signifier -= ySignifier;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 695          Signifier >= ySignifier)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L696](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L696)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 658          Signifier >>= uint256(-delta);
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 657          elta < 0)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L658](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L658)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 691          Signifier = (ySignifier - 1 >> uint256(delta - 1)) + 1;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 691          elta > 1)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L691](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L691)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 824          Signifier >>= 16_496 - xExponent;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 823          Exponent < 16_496)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L824](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L824)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 826          Signifier <<= xExponent - 16_496;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 825          Exponent > 16_496)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L826](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L826)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 834          Signifier >>= msb - 112;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 833          sb > 112)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L834](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L834)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 836          Signifier <<= 112 - msb;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 835          sb < 112)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L836](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L836)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 951          Signifier >>= msb - 112;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 950          sb > 112)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L951](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L951)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1080          esultSignifier = xExponent - 0x3FFF;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1078          Exponent >= 0x3FFF)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1080](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1080)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1158          Signifier <<= xExponent - 16_367;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1157          Exponent > 16_367)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1158](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1158)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1160          Signifier >>= 16_367 - xExponent;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1159          Exponent < 16_367)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1160](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1160)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 2014          esult <<= exponent - 16_495;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 2014          xponent > 16_495)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L2014](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L2014)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 2013          esult >>= 16_495 - exponent;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 2013          xponent < 16_495)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L2013](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L2013)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1591          Signifier <<= xExponent - 16_367;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1590          Exponent > 16_367)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1591](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1591)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1593          Signifier >>= 16_367 - xExponent;
```
 should be unchecked because there is check:<br>File: src/libraries/ABDKMathQuad.sol
```
 
Line: 1592          Exponent < 16_367)
```
<br><br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1593](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L1593)


</details>

# 


## Unnecessary Casting of Variables
- Severity: Gas Optimization
- Confidence: High

### Description
This detector scans for instances where a variable is casted to its own type. This is unnecessary and can be safely removed to improve code readability.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: src/functions/ConstantProduct.sol
```
 
Line: 40          erve = uint256((lpTokenSupply / n) ** n);


```
Unnecessary cast: `t256((lpTokenSupply / n) ** n);

` it cast to the same type.<br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct.sol#L40](https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct.sol#L40)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 844          eturn bytes16(

                    uint128(uint128((x ^ y) & 0x80000000000000000000000000000000) | xExponent << 112 | xSignifier)

                );
```
Unnecessary cast: `int128(uint128((x ^ y) & 0x80000000000000000000000000000000) | xExponent << 112 | xSignifier)
` it cast to the same type.<br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L844-L846](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L844-L846)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 959          eturn bytes16(

                    uint128(uint128((x ^ y) & 0x80000000000000000000000000000000) | xExponent << 112 | xSignifier)

                );
```
Unnecessary cast: `int128(uint128((x ^ y) & 0x80000000000000000000000000000000) | xExponent << 112 | xSignifier)
` it cast to the same type.<br>
[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L959-L961](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L959-L961)


</details>

# 


## Use a More Recent Version of Solidity
- Severity: Gas Optimization
- Confidence: High

### Description

`solc` frequently releases new compiler versions. Using an old version prevents access to new Solidity security checks.
Addition new version gain some gas boost.
We also recommend avoiding complex `pragma` statement.

<details>

<summary>
There are 27 instances of this issue:

</summary>

###
- File: src/Aquifer.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L3)


- File: src/Well.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L3)


- File: src/functions/ConstantProduct.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct.sol#L3)


- File: src/functions/ConstantProduct2.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L3)


- File: src/functions/ProportionalLPToken.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ProportionalLPToken.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ProportionalLPToken.sol#L3)


- File: src/functions/ProportionalLPToken2.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ProportionalLPToken2.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ProportionalLPToken2.sol#L3)


- File: src/interfaces/IAquifer.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/IAquifer.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/IAquifer.sol#L3)


- File: src/interfaces/IBeanstalkWellFunction.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/IBeanstalkWellFunction.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/IBeanstalkWellFunction.sol#L3)


- File: src/interfaces/IWell.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/IWell.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/IWell.sol#L3)


- File: src/interfaces/IWellErrors.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/IWellErrors.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/IWellErrors.sol#L3)


- File: src/interfaces/IWellFunction.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/IWellFunction.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/IWellFunction.sol#L3)


- File: src/interfaces/pumps/ICumulativePump.sol
```
 
Line: 3          pragma solidity =0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/pumps/ICumulativePump.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/pumps/ICumulativePump.sol#L3)


- File: src/interfaces/pumps/IInstantaneousPump.sol
```
 
Line: 3          pragma solidity =0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/pumps/IInstantaneousPump.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/pumps/IInstantaneousPump.sol#L3)


- File: src/interfaces/pumps/IMultiFlowPumpErrors.sol
```
 
Line: 3          pragma solidity =0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/pumps/IMultiFlowPumpErrors.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/pumps/IMultiFlowPumpErrors.sol#L3)


- File: src/interfaces/pumps/IPump.sol
```
 
Line: 3          pragma solidity =0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/pumps/IPump.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/interfaces/pumps/IPump.sol#L3)


- File: src/libraries/ABDKMathQuad.sol
```
 
Line: 6          ragma solidity ^0.8.0;

```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L6](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/ABDKMathQuad.sol#L6)


- File: src/libraries/LibBytes.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L3)


- File: src/libraries/LibBytes16.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes16.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes16.sol#L3)


- File: src/libraries/LibClone.sol
```
 
Line: 2          pragma solidity ^0.8.4;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibClone.sol#L2](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibClone.sol#L2)


- File: src/libraries/LibContractInfo.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L3)


- File: src/libraries/LibLastReserveBytes.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol#L3)


- File: src/libraries/LibMath.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibMath.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibMath.sol#L3)


- File: src/libraries/LibWellConstructor.sol
```
 
Line: 4          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L4](https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L4)


- File: src/pumps/MultiFlowPump.sol
```
 
Line: 3          pragma solidity ^0.8.17;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L3](https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L3)


- File: src/utils/Clone.sol
```
 
Line: 2          pragma solidity ^0.8.4;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/utils/Clone.sol#L2](https://github.com/code-423n4/2023-07-basin/blob/main/src/utils/Clone.sol#L2)


- File: src/utils/ClonePlus.sol
```
 
Line: 2          pragma solidity ^0.8.4;
```
 instead use `0.8.19`

[https://github.com/code-423n4/2023-07-basin/blob/main/src/utils/ClonePlus.sol#L2](https://github.com/code-423n4/2023-07-basin/blob/main/src/utils/ClonePlus.sol#L2)


- solc-0.8.17 is not recommended for deployment

</details>

# 