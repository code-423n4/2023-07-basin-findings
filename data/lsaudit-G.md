# [G-01] Ineffective loop in `_getIJ`

Current implementation of looking for `I` and `J` tokens is very ineffective. It always loops through the whole array, even when `I` and `J` are at the beginning of the array.

```
File: /src/Well.sol

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



`_getIJ` should be reimplemented to save gas (loop's iterations) for scenarios where `I` and `J` are at the beginning of the array. Firstly, loop once, until you find either `I` or `J` token (assume one of them has been found at index `X` of the array). `Break` from the loop and enter another loop from `X + 1` to the end of array. If you find either `I` or `J` again, `break` once more (as this means that both `I` and `J` are found).
This approach will save a lot of gas when I and J are at the beginning of the array. Otherwise, (in the worst case scenario: either I or J is the last element) - you will still loop over the whole array - as in the current implementation.