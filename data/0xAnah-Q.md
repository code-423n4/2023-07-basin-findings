# Non-Critical Issues


## [N-01] Unnecessary Code
The is no need to cache the token array since all that is needed in the function is just the length of the token array.
### 3 Instances
- https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L450
```
function getAddLiquidityOut(uint256[] memory tokenAmountsIn) external view returns (uint256 lpAmountOut) {
        IERC20[] memory _tokens = tokens();
        uint256[] memory reserves = _getReserves(_tokens.length);
        for (uint256 i; i < _tokens.length; ++i) {
            reserves[i] = reserves[i] + tokenAmountsIn[i];
        }
        lpAmountOut = _calcLpTokenSupply(wellFunction(), reserves) - totalSupply();
}
```
The above code be refactored to:
```
function getAddLiquidityOut(uint256[] memory tokenAmountsIn) external view returns (uint256 lpAmountOut) {
        uint256 _tokensLength = tokens().length;
        uint256[] memory reserves = _getReserves(_tokensLength);
        for (uint256 i; i < _tokensLength; ++i) {
            reserves[i] = reserves[i] + tokenAmountsIn[i];
        }
        lpAmountOut = _calcLpTokenSupply(wellFunction(), reserves) - totalSupply();
}
```
- https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L486
```
 function getRemoveLiquidityOut(uint256 lpAmountIn) external view returns (uint256[] memory tokenAmountsOut) {
        IERC20[] memory _tokens = tokens();
        uint256[] memory reserves = _getReserves(_tokens.length);
        uint256 lpTokenSupply = totalSupply();

        tokenAmountsOut = _calcLPTokenUnderlying(wellFunction(), lpAmountIn, reserves, lpTokenSupply);
}
```
The above code could be refactored to:
```
 function getRemoveLiquidityOut(uint256 lpAmountIn) external view returns (uint256[] memory tokenAmountsOut) {
        uint256 _tokensLength = tokens().length;
        uint256[] memory reserves = _getReserves(_tokensLength);
        uint256 lpTokenSupply = totalSupply();

        tokenAmountsOut = _calcLPTokenUnderlying(wellFunction(), lpAmountIn, reserves, lpTokenSupply);
}
```
- https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L578
```
function getRemoveLiquidityImbalancedIn(uint256[] calldata tokenAmountsOut)
        external
        view
        returns (uint256 lpAmountIn)
{
        IERC20[] memory _tokens = tokens();
        uint256[] memory reserves = _getReserves(_tokens.length);
        for (uint256 i; i < _tokens.length; ++i) {
            reserves[i] = reserves[i] - tokenAmountsOut[i];
        }
        lpAmountIn = totalSupply() - _calcLpTokenSupply(wellFunction(), reserves);
}
```
The above code be refactored to:
```
function getRemoveLiquidityImbalancedIn(uint256[] calldata tokenAmountsOut)
        external
        view
        returns (uint256 lpAmountIn)
{
        uint256 _tokensLength = tokens().length;
        uint256[] memory reserves = _getReserves(_tokensLength);
        for (uint256 i; i < _tokensLength; ++i) {
            reserves[i] = reserves[i] - tokenAmountsOut[i];
        }
        lpAmountIn = totalSupply() - _calcLpTokenSupply(wellFunction(), reserves);
}
```




## [N-02] Visibility should be set explicitly
The visibility of the variables should be set explicitly rather than defaulting to internal for clarity.
### 3 Instances 
- https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol

```
26:    uint256 constant ONE_WORD = 32;
27:    uint256 constant PACKED_ADDRESS = 20;
28:    uint256 constant ONE_WORD_PLUS_PACKED_ADDRESS = 52;

77:    uint256 constant LOC_AQUIFER_ADDR = 0;
```
The visibility of these constants could be set to private since they are not to be inherited. 

The code could be refactored to:
```
26:    uint256 private constant ONE_WORD = 32;
27:    uint256 private constant PACKED_ADDRESS = 20;
28:    uint256 private constant ONE_WORD_PLUS_PACKED_ADDRESS = 52;

77:    uint256 private constant LOC_AQUIFER_ADDR = 0;
```
- https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L25

```
 25:    mapping(address => address) wellImplementations;
```
The `wellImplementations` mapping could be made private since the contract is not inherited and there is a getter function for the mapping defined in the contract.

The code could be refactored to:
```
25:    mapping(address => address) private wellImplementations;
```

- https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L23
```
23:    uint256 constant EXP_PRECISION = 1e12;
```
The visibility of these constants could be set to private since they are not to be inherited.

The code could be refactored to:
```
23:    uint256 private constant EXP_PRECISION = 1e12;
```

## [N-03] The if statement can be refactored to the ternary
The normal if / else statement can be written in a shorthand way using the ternary operator. It increases readability and reduces the number of lines of code.
- https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L40-#L52
```
if (immutableData.length > 0) {
    if (salt != bytes32(0)) {
        well = implementation.cloneDeterministic(immutableData, salt);
    } else {
        well = implementation.clone(immutableData);
    }
} else {
    if (salt != bytes32(0)) {
        well = implementation.cloneDeterministic(salt);
    } else {
        well = implementation.clone();
    }    
}
```
In the code above the nested if-else statements could be re-structured to ternary therefore the refactored code would be:

```
if (immutableData.length > 0) {

    well = salt != bytes32(0) ? implementation.cloneDeterministic(immutableData, salt) : implementation.clone(immutableData);
    
} else {

    well = salt != bytes32(0) ? implementation.cloneDeterministic(salt) : implementation.clone();
}
```