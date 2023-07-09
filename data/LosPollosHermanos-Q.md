# [LOW] No checks if `j` is `0` or `1` in `CP2` `calcReservesAtRatioSwap` , `calcReserve` & `calcReserveAtRatioLiquidity`

# Impact 
If a `Well` holds more than two tokens and uses `ConstantProduct2` as its `Well Function` calling the function with `j != 1 && j != 0` will make the reserves inacurate.

If reserves are only a 2 arguments array than this will result in an error.

# Proof of Concept
https://github.com/code-423n4/2023-07-basin/blob/c1b72d4e372a6246e0efbd57b47fb4cbb5d77062/src/functions/ConstantProduct2.sol#L58
    
https://github.com/code-423n4/2023-07-basin/blob/c1b72d4e372a6246e0efbd57b47fb4cbb5d77062/src/functions/ConstantProduct2.sol#L79

https://github.com/code-423n4/2023-07-basin/blob/c1b72d4e372a6246e0efbd57b47fb4cbb5d77062/src/functions/ConstantProduct2.sol#L92

## Solution
Add `require(j == 0 || j == 1);` in all of the above functions.

# [NON-CRITICAL] If 'Well' stores more tokens than the reserves `swapFrom` won't revert using a token with a fee on transfer

## Impact
Not having a fee on transfer is a prerequisite for a token to be swapped using `swapFrom` and is checked in `_setReserves`; however, if the amounts of tokens stored in a `Well` are higher than reserves than this transaction may not revert.

## Proof of Concept
We consider `swapFrom` function: https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L186

Let `token1` and `token2` be arbitrary tokens being swapped. Let `T1`, `T2` be numbers of this tokens stored by the `Well` and `R1`, `R2` reserves of this tokens.
1. Let this be a initial status of the `Well`:
   T1 = 100; R1 = 50
   T2 = 100; R2 = 50
 
2. 5 tokens of `token1` are added but it has fee on transfer 25% 
   T1 = 104; R1 = 55
   T2 = 100; R2 = 50
    
In this case the assertions https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L634 pass.

## Solution
Use `swapFromFeeOnTransfer` for every transfer.

# [Non-Critical] AMM functionality is limited with the `_calcReserve()` functions
## Impact
AMMs which are self-balancing (to stabilise prices) cannot be implemented as Basin only allows only two reserves to be modified in a swap (`tokenFrom` and `tokenTo`).

## Proof of Concept
https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L701

## Solution 
Allow `wellFunction` to modify all reserves.
