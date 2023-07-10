# Unsafe usage with `abi.encodePacked`


## Impact
Based on the description in [non-standard-packed-mode](https://docs.soliditylang.org/en/v0.8.17/abi-spec.html?highlight=collisions#non-standard-packed-mode) the  `abi.encodePacked` methods will not padding when the given arguments are dynamic types.

```
If you use keccak256(abi.encodePacked(a, b)) and both a and b are dynamic types, it is easy to craft collisions in the hash value by moving parts of a into b and vice-versa. More specifically, abi.encodePacked("a", "bc") == abi.encodePacked("ab", "c")
```

## Proof of Concept
The following code may suffer the collison:
```solidity=44
            packedPumps = abi.encodePacked(
                packedPumps,            // previously packed pumps
                _pumps[i].target,       // pump address
                _pumps[i].data.length,  // pump data length
                _pumps[i].data          // pump data (bytes)
            );
```
`target`, `data.length` and `data` can always generate an alternative to create such collision since it's purely based on the input.

```solidity=52
        immutableData = abi.encodePacked(
            _aquifer,                   // aquifer address
            _tokens.length,             // number of tokens
            _wellFunction.target,       // well function address
            _wellFunction.data.length,  // well function data length
            _pumps.length,              // number of pumps
            _tokens,                    // tokens array
            _wellFunction.data,         // well function data (bytes)
            packedPumps                 // packed pumps (bytes)
        );
```
`_tokens.length`, `_wellFunction.target`, `_wellFunction.data.length` and `_pumps.length` may also generate such equivalent.

## Tools Used

## Recommended Mitigation Steps
Recommend using `abi.encode` instead
