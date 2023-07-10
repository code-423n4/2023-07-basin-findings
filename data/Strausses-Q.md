Since there is no code presented for a function calcLpTokenSupply()
/src/interfaces/IwellFunction.sol

https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L436

Be careful since there could be a First depositor issue, same as here 
https://solodit.xyz/issues/h-03-first-depositor-can-break-minting-of-shares-code4rena-caviar-caviar-contest-git

Recommendation, if this issue occurs, is 
Uniswap V2 solved this problem by sending the first 1000 LP tokens to the zero address. The same can be done in this case i.e. when lpTokenSupply == 0, send the first min liquidity LP tokens to the zero address to enable share dilution.
https://github.com/Uniswap/v2-core/blob/master/contracts/UniswapV2Pair.sol#L119-L124

