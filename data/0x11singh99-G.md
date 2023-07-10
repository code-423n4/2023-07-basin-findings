### Gas Optimizations List

| Number | Optimization Details                                                                                      | Instances |
| :----: | :-------------------------------------------------------------------------------------------------------- | :-------: |
| [G-01] | Do not calculate constant variables                                                                       |     6     |
| [G-02] | Check recipient for address(0) to fail early rather than checking inside function call                    |     4     |
| [G-03] | Write for loops in more gas efficient way                                                                 |     6     |
| [G-04] | Memory variables should be cached in stack(if possible) variables rather than re-reading them from memory |     5     |
| [G-05] | Function calls can be cached rather than re calling from same function with same value will save gas      |     2     |

| [G-06] | Use unchecked{} whenever underflow/overflow not possible saves 130 gas each time | 1 |
Total 6 issues

## [G-01] Do not calculate constant variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas each time of use.

_Total 6 instances - 1 files:_

### Instance#1-6 : Assign direct simple constant value after calculating off chain

```solidity
File : /src/Well.sol

29: bytes32 constant RESERVES_STORAGE_SLOT = bytes32(uint256(keccak256("reserves.storage.slot")) - 1);
         ...
78:    uint256 constant LOC_TOKENS_COUNT = LOC_AQUIFER_ADDR + PACKED_ADDRESS;
79:    uint256 constant LOC_WELL_FUNCTION_ADDR = LOC_TOKENS_COUNT + ONE_WORD;
80:    uint256 constant LOC_WELL_FUNCTION_DATA_LENGTH = LOC_WELL_FUNCTION_ADDR + PACKED_ADDRESS;
81:    uint256 constant LOC_PUMPS_COUNT = LOC_WELL_FUNCTION_DATA_LENGTH + ONE_WORD;
82:    uint256 constant LOC_VARIABLE = LOC_PUMPS_COUNT + ONE_WORD;

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L29C5-L30C1
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L78C4-L82C64

## [G-02] Check recipient for address(0) to fail early rather than checking inside function call

_Total 4 instances - 1 files:_

### Instance#1 : `recipient` can be checked for `address(0)` in starting of external function to fail early.

```solidity
File : /src/Well.sol

195: amountOut = _swapFrom(fromToken, toToken, amountIn, minAmountOut, recipient);
       ...
212:  amountOut = _swapFrom(fromToken, toToken, amountIn, minAmountOut, recipient);
       ...
398:  lpAmountOut = _addLiquidity(tokenAmountsIn, minLpAmountOut, recipient, false);
     ...
407:  lpAmountOut = _addLiquidity(tokenAmountsIn, minLpAmountOut, recipient, true);
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L195
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L212
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L398
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L407

## [G-03] Write for loops in more gas efficient way

Loop can be written in more gas efficient way, by

1. taking length in stack variable,
2. using unchecked{++i} with pre increment
3. not re-initializing to 0 since it is default
4. Using <= is cheaper than <

_6 instances - 1 file:_

### Instance#1:

```solidity
File: src/pumps/MultiFlowPump.sol
301:    for (uint256 i; i < cumulativeReserves.length; ++i) {
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L301

Recommended Changes: Convert above for loop to this for making gas efficient saves more than 130 Gas per iteration

```diff
-      for (uint256 i; i < cumulativeReserves.length; ++i) {
        ...

        }

+  uint256 _size = cumulativeReserves.length - 1;
+  for (uint256 i; i <= _size ; ) {

              ...
+            unchecked {
+                ++i;
+            }
        }

```

### Instance#2:

```solidity
File: src/pumps/MultiFlowPump.sol
322:   for (uint256 i; i < byteCumulativeReserves.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L322
Update like above

### Instance#3:

```solidity
File: /src/Well.sol
36:   for (uint256 i; i < _tokens.length - 1; ++i) {
37:            for (uint256 j = i + 1; j < _tokens.length; ++j) {

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L36C6-L37C63
Update like first instance

### Instance#4:

```solidity
File: /src/Well.sol
101:     for (uint256 i; i < _pumps.length; i++) {

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L101C7-L101C50
Update like first instance

### Instance#5:

```solidity
File: /src/Well.sol
363:      for (uint256 i; i < _tokens.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L363C8-L363C51
Update like first instance

### Instance#6:

```solidity
File: /src/Well.sol
382:  for (uint256 i; i < _tokens.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L382C8-L382C51
Update like first instance

Note: These types of instances are many more in the Well.sol file so make sure to update them
for gas efficiency.

## [G‑04] Memory variables should be cached in stack(if possible) variables rather than re-reading them from memory

It saves extra MLOAD opcode and saves gas

_5 instances - 2 files:_

### Instance#1-2: `pumpState.lastTimestamp` and `reserves[i]` can be cached

```solidity
File: src/pumps/MultiFlowPump.sol

72: function update(uint256[] calldata reserves, bytes calldata) external {
        ...
83:    if (pumpState.lastTimestamp == 0){
        ...
109:   uint256 deltaTimestamp = _getDeltaTimestamp(pumpState.lastTimestamp);
        ...
119:    pumpState.lastReserves[i], (reserves[i] > 0 ? reserves[i] : 1).fromUIntToLog2(), blocksPassed
        ...
140:   }

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L83
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L109
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L119

### Instance#3: `reserves[i]` can be cached to stack variable

```solidity
File: src/pumps/MultiFlowPump.sol

153:   if (reserves[i] == 0) return;
154:         byteReserves[i] = reserves[i].fromUIntToLog2();

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L153C1-L154C60

### Instance#4-5: `tokenAmountsOut[i]` and `minTokenAmountsOut[i]` can be cached to stack variable

```solidity
 File: src/Well.sol
 473:    for (uint256 i; i < _tokens.length; ++i) {
            if (tokenAmountsOut[i] < minTokenAmountsOut[i]) {
                revert SlippageOut(tokenAmountsOut[i], minTokenAmountsOut[i]);
            }
            _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);
            reserves[i] = reserves[i] - tokenAmountsOut[i];
 479:       }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L473C6-L479C10

Note: These types of instances are many more in the Well.sol file so make sure to update them
for gas efficiency.

## [G-05] Function calls can be cached rather than re calling from same function with same value will save gas

_2 instances - 1 file:_

### Instance#1 : LibContractInfo.getSymbol(address(\_tokens[i])) can be cached instead of 2 time reading for each iteration

```solidity
File : /src/libraries/LibWellConstructor.sol

73:   for (uint256 i = 1; i < _tokens.length; ++i) {
74:           name = string.concat(name, ":", LibContractInfo.getSymbol(address(_tokens[i])));
75:          symbol = string.concat(symbol, LibContractInfo.getSymbol(address(_tokens[i])));
76:        }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L73C8-L76C10

## [G-06] Use unchecked{} whenever underflow/overflow not possible saves 130 gas each time

Because of above check, it can be decided that underflow/overflow not possible.

_1 instance - 1 file:_

### Instance#1 : `i-1` can be unchecked due to line 93 for () condition initialization and increment

```solidity
File:

 93:         for (uint256 i = 3; i <= n; ++i) {
                // `iByte` is the byte position for the current slot:
                // i        3 4 5 6
                // iByte    1 1 2 2
 97:               iByte = ((i - 1) / 2) * 32;
                ...
          }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol#L93C9-L97C42
