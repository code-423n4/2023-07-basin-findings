## Gas optimizations list

| Number     | Issue                                                     | Instances |
|------------|-----------------------------------------------------------|-----------|
| [G-01]     | Use constants instead of `type(uintx).max`                | 5         |
| [G-02]     | Do not calculate constants                                | 6         |
| [G-03]     | `_getIJ` function can exit early                          | 1         |
| [G-04]     | Use bit shifting instead of multiplying by powers of 2    | 3         |
| [G-05]     | Combine multiple transfers into singular transfer         | 1         |
| [G-06]     | Rearrange `require`/`revert` statements                   | 1         |



### [G-01] Use constants instead of `type(uintx).max`

Using `type(uintx).max` such as `type(uint128).max` incurs more gas in distribution and for each transaction. Constants (such as `340282366920938463463374607431768211455` for `type(uint128).max`) should be used instead.

*There are 5 instances of this issue:*
```solidity
File: LibBytes.sol
  40:             require(reserves[0] <= type(uint128).max, "ByteStorage: too large"); // @audit gas dont express as type(uintx).max
  41:             require(reserves[1] <= type(uint128).max, "ByteStorage: too large");
  49:                 require(reserves[2 * i] <= type(uint128).max, "ByteStorage: too large");
  50:                 require(reserves[2 * i + 1] <= type(uint128).max, "ByteStorage: too large");
  63:                 require(reserves[reserves.length - 1] <= type(uint128).max, "ByteStorage: too large");
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L40-L41
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L49-L50
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L63

### [G-02] Do not calculate constants

The implementation of constant variables (replacements at compile-time) leads to the recomputation of an expression assigned to a constant variable each time the variable is used, wasting gas. Therefore constants should not involve calculations, rather the final value. For clarity, use comments to show where that value is being derived from.

*There are 6 instances of this issue:*

```solidity
File: Well.sol
77:     uint256 constant LOC_AQUIFER_ADDR = 0;
78:     uint256 constant LOC_TOKENS_COUNT = LOC_AQUIFER_ADDR + PACKED_ADDRESS; // @audit gas - all of these can be precalculated, expressed not as expressions, use comments for clarity
79:     uint256 constant LOC_WELL_FUNCTION_ADDR = LOC_TOKENS_COUNT + ONE_WORD;
80:     uint256 constant LOC_WELL_FUNCTION_DATA_LENGTH = LOC_WELL_FUNCTION_ADDR + PACKED_ADDRESS;
81:     uint256 constant LOC_PUMPS_COUNT = LOC_WELL_FUNCTION_DATA_LENGTH + ONE_WORD;
82:     uint256 constant LOC_VARIABLE = LOC_PUMPS_COUNT + ONE_WORD;
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L77-L82

This includes `keccak256` hashes:

```solidity
File: Well.sol
29:     bytes32 constant RESERVES_STORAGE_SLOT = bytes32(uint256(keccak256("reserves.storage.slot")) - 1); // @audit gas - calculate hashes, dont make constants be expressions
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L29


### [G-03] `_getIJ` function can exit early, to prevent unnecessary further looping

In the `_getIJ` function, if both `iToken` and `jToken` are found before the loop ends, you can exit early to avoid further checking values, which may save gas depending on the elements in the array.

*There is 1 instance of this issue:*

Original:
```solidity
File: Well.sol
    function _getIJ(
        IERC20[] memory _tokens,
        IERC20 iToken,
        IERC20 jToken
    ) internal pure returns (uint256 i, uint256 j) {
        bool foundI = false;
        bool foundJ = false;

        for (uint256 k; k < _tokens.length; ++k) {
            if (iToken == _tokens[k]) {
                i = k;
                foundI = true;
            } else if (jToken == _tokens[k]) {
                j = k;
                foundJ = true;
            }
        }

        if (!foundI) revert InvalidTokens();
        if (!foundJ) revert InvalidTokens();
    }
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L730-L750

Diff:

```diff
File: Well.sol
 function _getIJ(
     IERC20[] memory _tokens,
     IERC20 iToken,
     IERC20 jToken
 ) internal pure returns (uint256 i, uint256 j) {
-    bool foundI = false;
-    bool foundJ = false;
 
     for (uint256 k; k < _tokens.length; ++k) {
-        if (iToken == _tokens[k]) {
-            i = k;
-            foundI = true;
-        } else if (jToken == _tokens[k]) {
-            j = k;
-            foundJ = true;
+        if (iToken == _tokens[k] && jToken == _tokens[k]) {
+            return; // Exit early if both tokens are found
         }
        }
    }
        if (!foundI) revert InvalidTokens();
        if (!foundJ) revert InvalidTokens();
    }
```

### [G-04] Use bit shifting instead of multiplying by powers of 2

*Note: This is in automated findings, but some instances were missed. Below contains the full list of instances and the ones that were missed.*

In `Well.sol`, `ONE_WORD` is defined as `32`. Throughout the code other numbers are multiplied by `ONE_WORD`, but it is more gas efficient to left bit shift by 5 positions.

There are also other instances where the integar is directly multiplied by a power of 2. This should be replaced by bit shifting to save gas.

*Missed: 3 instances of this issue:*

```solidity
File: Well.sol
90:         uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD;
98:         uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD + wellFunctionDataLength();
174:         uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD + wellFunctionDataLength();
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L90
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L98
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L174


*Total (including missed): 15 instances of this issue:*

```solidity
File: Well.sol
90:         uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD;
98:         uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD + wellFunctionDataLength();
174:         uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD + wellFunctionDataLength();
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L90
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L98
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L174
```solidity
File: LibBytes.sol
22:         uint256 index = i * 32;
49:                 require(reserves[2 * i] <= type(uint128).max, "ByteStorage: too large");
50:                 require(reserves[2 * i + 1] <= type(uint128).max, "ByteStorage: too large");
51:                 iByte = i * 64;
64:                 iByte = maxI * 64;
96:             iByte = (i - 1) / 2 * 32;
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L22
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L49
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L50
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L51
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L64
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L96
```solidity
File: LibBytes16.sol
29:                 iByte = i * 64;
41:                 iByte = maxI * 64;
73:             iByte = (i - 1) / 2 * 32;
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes16.sol#L29
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes16.sol#L41
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes16.sol#L73
```solidity
File: LibLastReserveBytes.sol
42:                 iByte = i * 64;
54:                 iByte = maxI * 64;
97:                 iByte = (i - 1) / 2 * 32;
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol#L42

### [G-05] Combine multiple transfers into singular transfer

In the `skim` function, `skimAmounts` array is looped through, performing a transfer if the element is greater than 0. However, it is more gas efficient to combine the multiple transfers into a singular transfer at the end.

See the suggestion below:

*There is 1 instances of this issue:*
```solidity
File: Well.sol
600:     /**
601:      * @dev Transfer excess tokens held by the Well to `recipient`.
602:      */
603:     function skim(address recipient) external nonReentrant returns (uint256[] memory skimAmounts) {
604:         IERC20[] memory _tokens = tokens();
605:         uint256[] memory reserves = _getReserves(_tokens.length);
606:         skimAmounts = new uint256[](_tokens.length);
607:         for (uint256 i; i < _tokens.length; ++i) {
608:             skimAmounts[i] = _tokens[i].balanceOf(address(this)) - reserves[i];
609:             if (skimAmounts[i] > 0) {
610:                 _tokens[i].safeTransfer(recipient, skimAmounts[i]);
611:             }
612:         }
613:     }
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L600-L613

Can be rewritten as:
```solidity
File: Well.sol
600:     /**
601:      * @dev Transfer excess tokens held by the Well to `recipient`.
602:      */
603:     function skim(address recipient) external nonReentrant returns (uint256[] memory skimAmounts) {
604:         IERC20[] memory _tokens = tokens();
605:         uint256[] memory reserves = _getReserves(_tokens.length);
606:         uint256 amountToTransfer;
607:         skimAmounts = new uint256[](_tokens.length);
608:         for (uint256 i; i < _tokens.length; ++i) {
609:             skimAmounts[i] = _tokens[i].balanceOf(address(this)) - reserves[i];
610:             if (skimAmounts[i] > 0) {
611:                 amountToTransfer += skimAmounts[i];
612:             }
613:         }
614:         if (amountToTransfer > 0){ // can use != to save gas as well
615:             _tokens[i].safeTransfer(recipient, skimAmounts[i]);
616:         }
617:     }
```

Diff:

```diff
File: Well.sol
 600:     /**
 601:      * @dev Transfer excess tokens held by the Well to `recipient`.
 602:      */
 603:     function skim(address recipient) external nonReentrant returns (uint256[] memory skimAmounts) {
 604:         IERC20[] memory _tokens = tokens();
 605:         uint256[] memory reserves = _getReserves(_tokens.length);
-606:         skimAmounts = new uint256[](_tokens.length);
-607:         for (uint256 i; i < _tokens.length; ++i) {
-608:             skimAmounts[i] = _tokens[i].balanceOf(address(this)) - reserves[i];
-609:             if (skimAmounts[i] > 0) {
-610:                 _tokens[i].safeTransfer(recipient, skimAmounts[i]);
-611:             }
-612:         }
-613:     }
+606:         uint256 amountToTransfer;
+607:         skimAmounts = new uint256[](_tokens.length);
+608:         for (uint256 i; i < _tokens.length; ++i) {
+609:             skimAmounts[i] = _tokens[i].balanceOf(address(this)) - reserves[i];
+610:             if (skimAmounts[i] > 0) {
+611:                 amountToTransfer += skimAmounts[i];
+612:             }
+613:         }
+614:         if (amountToTransfer > 0){ // can use != to save gas as well
+615:             _tokens[i].safeTransfer(recipient, skimAmounts[i]);
+616:         }
+617:     }
```

### [G-06] Rearrange `require`/`revert` statements to save gas

It is more gas efficient to fail early, so `require`/`revert` statements checking for example, function parameters should be placed at the start.

See suggestions below:

*There is 1 instance of this issue:*

In the constructor in `MultiFlowPump` the parameters should be checked before modifying state variables, to save gas in the case that it reverts.

```solidity
File: MultiFlowPump.sol
54:     constructor(bytes16 _maxPercentIncrease, bytes16 _maxPercentDecrease, uint256 _blockTime, bytes16 _alpha) {
55:         LOG_MAX_INCREASE = ABDKMathQuad.ONE.add(_maxPercentIncrease).log_2();
56:         // _maxPercentDecrease <= 100%
57:         if (_maxPercentDecrease > ABDKMathQuad.ONE) {
58:             revert InvalidMaxPercentDecreaseArgument(_maxPercentDecrease);
59:         }
60:         LOG_MAX_DECREASE = ABDKMathQuad.ONE.sub(_maxPercentDecrease).log_2();
61:         BLOCK_TIME = _blockTime;
62: 
63:         // ALPHA <= 1
64:         if (_alpha > ABDKMathQuad.ONE) {
65:             revert InvalidAArgument(_alpha);
66:         }
67:         ALPHA = _alpha;
68:     }
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L54-L68

Should be rearranged to:

```diff
File: MultiFlowPump.sol
 54:     constructor(bytes16 _maxPercentIncrease, bytes16 _maxPercentDecrease, uint256 _blockTime, bytes16 _alpha) {
-55:         LOG_MAX_INCREASE = ABDKMathQuad.ONE.add(_maxPercentIncrease).log_2();
-56:         // _maxPercentDecrease <= 100%
-57:         if (_maxPercentDecrease > ABDKMathQuad.ONE) {
-58:             revert InvalidMaxPercentDecreaseArgument(_maxPercentDecrease);
-59:         }
-60:         LOG_MAX_DECREASE = ABDKMathQuad.ONE.sub(_maxPercentDecrease).log_2();
-61:         BLOCK_TIME = _blockTime;
-62:
-63:         // ALPHA <= 1
-64:         if (_alpha > ABDKMathQuad.ONE) {
-65:             revert InvalidAArgument(_alpha);
-66:         }
-67:         ALPHA = _alpha;
-68:     }
+55:         // _maxPercentDecrease <= 100%
+56:         if (_maxPercentDecrease > ABDKMathQuad.ONE) {
+57:             revert InvalidMaxPercentDecreaseArgument(_maxPercentDecrease);
+58:         }
+59:         // ALPHA <= 1
+60:         if (_alpha > ABDKMathQuad.ONE) {
+61:             revert InvalidAArgument(_alpha);
+62:         }
+63:
+64:         LOG_MAX_INCREASE = ABDKMathQuad.ONE.add(_maxPercentIncrease).log_2();
+65:         LOG_MAX_DECREASE = ABDKMathQuad.ONE.sub(_maxPercentDecrease).log_2();
+66:         BLOCK_TIME = _blockTime;
+67:
+68:         ALPHA = _alpha;
+69:     }
```

