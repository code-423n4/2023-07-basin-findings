# [G-01] Use `Do While` of instead `for` in some functions

Using `do while` instead of `for` in some functions may save some gas

## Example

### Before
```sol
for(uint256 i;i < _tokens.length;++i;) {
    reserves[i] = _tokens[i].balanceOf(address(this));
}
```


### After
```
uint256 i;
do {
    reserves[i] = _tokens[i].balanceOf(address(this));
    unchecked { ++i; }
} while(i < _tokens.length);

```

## Functions Links
https://github.com/code-423n4/2023-07-basin/blob/c1b72d4e372a6246e0efbd57b47fb4cbb5d77062/src/Well.sol#L382
https://github.com/code-423n4/2023-07-basin/blob/c1b72d4e372a6246e0efbd57b47fb4cbb5d77062/src/Well.sol#L452
https://github.com/code-423n4/2023-07-basin/blob/c1b72d4e372a6246e0efbd57b47fb4cbb5d77062/src/Well.sol#L593
https://github.com/code-423n4/2023-07-basin/blob/c1b72d4e372a6246e0efbd57b47fb4cbbd77062/src/Well.sol#L632
https://github.com/code-423n4/2023-07-basin/blob/c1b72d4e372a6246e0efbd57b47fb4cbb5d77062/src/Well.sol#L760
