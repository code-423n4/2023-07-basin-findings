# Gas Optimization 

## [G-01 ] The result of a function call should be cached rather than re-calling the function
117` return _getArgAddress(LOC_AQUIFER_ADDR);`
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L117

145: ` return _getArgUint256(LOC_TOKENS_COUNT);`
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L145

152: `return _getArgAddress(LOC_WELL_FUNCTION_ADDR);`
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L152

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L343

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L350

and so on

## [G-02] For uint use != 0 instead of > 0
54: `if (initFunctionCall.length > 0) {`
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L54

## [G-03] unchecked { ++i } is more gas efficient than i++ for loops
```
	for (uint256 i; i < _tokens.length - 1; ++i) {
            for (uint256 j = i + 1; j < _tokens.length; ++j) {
```
https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L36C9-L37C63

363: `for (uint256 i; i < _tokens.length; ++i) {`
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L363

more of this is in below links 
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes16.sol

## [G-04] Use calldata instead of memory for function arguments that do not get mutated
31: ` function init(string memory name, string memory symbol) public initializer {`
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L31

84: `function tokens() public pure returns (IERC20[] memory ts) {`
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L84

114: `function wellData() public pure returns (bytes memory) {}`
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L114

## [G-5 ]  x += y costs more gas than x = x + y for state variables
```
 	   dataLoc += PACKED_ADDRESS;
            pumpDataLength = _getArgUint256(dataLoc);
            dataLoc += ONE_WORD;
            _pumps[i].data = _getArgBytes(dataLoc, pumpDataLength);
            dataLoc += pumpDataLength;
```
https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L103C2-L107C39

226:`reserves[i] += amountIn;`
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L226
