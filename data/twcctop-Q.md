https://github.com/code-423n4/2023-07-basin/blame/84dd4472a951bfd66f4e92abd8231aa4d00f7398/src/Well.sol#L224
` _swapFrom`  fromToken and toToken should not be the same. Actually, there is no check related to the same from and to token. 
 But in `_getIj` same from、to token may revert because only one index can be found after all better check is recommended.




https://github.com/code-423n4/2023-07-basin/blob/84dd4472a951bfd66f4e92abd8231aa4d00f7398/src/Well.sol#L276C30-L276

```

 reserves[j] -= amountOut;

``` 
There is no check wheather   reserves[j] > amountOut，better check in recommended. 