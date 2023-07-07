In well.so init function.

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
