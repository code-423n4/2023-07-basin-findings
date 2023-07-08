# GAS OPTIMIZATION REPORT  

## [G-01] UNUSED IF STAMENT  
in The _addLiquidity function there is a if stament which is checking for 0 value and continue the execution of the function.  

```  
file: /src/Well.sol
function _addLiquidity(uint256[] memory tokenAmountsIn,uint256 minLpAmountOut, address recipient, bool feeOnTransfer     ) internal returns (uint256 lpAmountOut) {                 
               ...   
               if (tokenAmountsIn[i] == 0) continue;                 
               ...                 
               if (tokenAmountsIn[i] == 0) continue;                  
               ...     
} 

``` 
https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L413C5-L445C1 

we can perfectly omit this if statement due the contract is no doing nothin with it saving gas in the function.