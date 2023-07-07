### Non-Critical Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [N-01] |In `Well.sol init function`,use more efficient algorithms.| 1 |
| [N-02] |Add a break to the for in the `_getIJ` function in `Well.sol`.|1|
| [N-03] |Array length not checked. |2|

Total 3 issues

### [L-01] In `Well.sol init function`,use more efficient algorithms.

https://github.com/code-423n4/2023-07-basin/blob/c1b72d4e372a6246e0efbd57b47fb4cbb5d77062/src/Well.sol#L31

There are nested for loops that determine if there are duplicate elements in the array.
Time complexity is O(n^2).
```solidity
     for (uint256 i; i < _tokens.length - 1; ++i) {
            for (uint256 j = i + 1; j < _tokens.length; ++j) {
                if (_tokens[i] == _tokens[j]) {
                    revert DuplicateTokens(_tokens[i]);
                }
            }
       }
```

It can be replaced with a more efficient algorithm with O(n) time complexity:

```solidity
        function hasDuplicate(IERC20[] memory arr) public pure returns (bool) {
            mapping(uint256 => bool) seen;
            for (uint256 i = 0; i < arr.length; i++) {
                if (seen[arr[i]]) {
                    return true;
                }
                seen[arr[i]] = true;
            }
            return false;
        }
```

### [L-02] Add a break to the for in the `_getIJ` function in `Well.sol`.

https://github.com/code-423n4/2023-07-basin/blob/c1b72d4e372a6246e0efbd57b47fb4cbb5d77062/src/Well.sol#L730

If I and J are found, there is no need to continue the loop and we can add a break to break out of the loop.

before:
```
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

after:
```
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
            } else if (foundI && foundJ) {  // <---add break
                break;
            }
        }

        if (!foundI) revert InvalidTokens();
        if (!foundJ) revert InvalidTokens();
    }
```

### [L-03] Array length not checked.

https://github.com/code-423n4/2023-07-basin/blob/c1b72d4e372a6246e0efbd57b47fb4cbb5d77062/src/Well.sol#L413C1-L413C1

`tokenAmountsIn` is input by the external, and the array length is not checked during the for loop, and the array may be out of bounds.

```
function _addLiquidity(
        uint256[] memory tokenAmountsIn,
        uint256 minLpAmountOut,
        address recipient,
        bool feeOnTransfer
    ) internal returns (uint256 lpAmountOut) {
        IERC20[] memory _tokens = tokens();
        uint256[] memory reserves = _updatePumps(_tokens.length);

        if (feeOnTransfer) {
            for (uint256 i; i < _tokens.length; ++i) {
                if (tokenAmountsIn[i] == 0) continue;  // <---tokenAmountsIn may be out of bounds
                tokenAmountsIn[i] = _safeTransferFromFeeOnTransfer(_tokens[i], msg.sender, tokenAmountsIn[i]);
                reserves[i] = reserves[i] + tokenAmountsIn[i];
            }
        } 
        .....
    }
```

https://github.com/code-423n4/2023-07-basin/blob/c1b72d4e372a6246e0efbd57b47fb4cbb5d77062/src/Well.sol#L460C2-L460C2
`minTokenAmountsOut`is input by the external, and the array length is not checked during the for loop, and the array may be out of bounds.

```
  function removeLiquidity(
        uint256 lpAmountIn,
        uint256[] calldata minTokenAmountsOut,
        address recipient,
        uint256 deadline
    )
        external
        nonReentrant
        expire(deadline)
        returns (uint256[] memory tokenAmountsOut)
    {
        IERC20[] memory _tokens = tokens();
        uint256[] memory reserves = _updatePumps(_tokens.length);
        uint256 lpTokenSupply = totalSupply();

        tokenAmountsOut = new uint256[](_tokens.length);
        _burn(msg.sender, lpAmountIn);
        tokenAmountsOut = _calcLPTokenUnderlying(
            wellFunction(),
            lpAmountIn,
            reserves,
            lpTokenSupply
        );
        for (uint256 i; i < _tokens.length; ++i) {
            if (tokenAmountsOut[i] < minTokenAmountsOut[i]) {  // <---minTokenAmountsOut may be out of bounds
                revert SlippageOut(tokenAmountsOut[i], minTokenAmountsOut[i]);
            }
            _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);
            reserves[i] = reserves[i] - tokenAmountsOut[i];
        }

        _setReserves(_tokens, reserves);
        emit RemoveLiquidity(lpAmountIn, tokenAmountsOut, recipient);
    }
```