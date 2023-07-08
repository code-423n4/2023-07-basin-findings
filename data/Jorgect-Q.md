#GAS OPTIMIZATION REPORT

##[G-01] UNUSED IF STAMENT

in The _addLiquidity function there is a if stament which is checking for 0 value and continue the execution of the function

```
 function _addLiquidity(
        uint256[] memory tokenAmountsIn,
        uint256 minLpAmountOut,
        address recipient,
        bool feeOnTransfer
    ) internal returns (uint256 lpAmountOut) {
                ...
                if (tokenAmountsIn[i] == 0) continue;
                ...
                if (tokenAmountsIn[i] == 0) continue; 
                ...
    }
```

we can perfectly omit this if statement due the contract is no doing nothin with it saving gas in the function.


