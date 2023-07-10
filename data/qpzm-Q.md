##  1. The first LP provider might mint 0 Well token by mistake.

###  proof of concept
First, comment out the line 436 in `TestHelper.sol`.
https://github.com/code-423n4/2023-07-basin/blob/main/test/TestHelper.sol#L107
```solidity
function setupWell(Call memory _wellFunction, Call[] memory _pumps, IERC20[] memory _tokens) internal {
    // ...
    // @audit Comment out the line 436
    // Add initial liquidity from TestHelper
    // addLiquidityEqualAmount(address(this), initialLiquidity);
}
```

`Well._addLiquidity` does not revert when `ConstantProduct2.calcLpTokenSupply` returns 0.
Since 0 * 100 is 0, we mint 0 LP token.
Add this test in `Well.AddLiquidity.t.sol`.
```solidity
function test_addLiquidity0() public {
    uint256[] memory amounts = new uint256[](tokens.length);
    amounts[0] = 100;
    well.addLiquidity(amounts, 0, user, type(uint256).max);

    Balances memory userBalance = getBalances(user, well);
    assertEq(userBalance.lp, 0);
}
```

###  recommended mitigation steps
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L441
```diff
+ require(lpAmountOut > 0, "lpAmountOut should be nonzero");
_mint(recipient, lpAmountOut);

```

##  2. `Well.decimals()` 18 is misleading

### Description
Curve 3pool uses 18 decimals and 1e18 LP tokens is almost equivalent to 1DAI according to the return value of `get_virtual_price()`.
https://github.com/curvefi/curve-contract/blob/master/contracts/tokens/CurveTokenV3.vy#L50-L56
https://github.com/curvefi/curve-contract/blob/master/contracts/pools/3pool/StableSwap3Pool.vy#L229

`UniswapV2Pair` uses 18 decimals for LP tokens and 1 UniswapV2Pair token is equivalent to the sum of 1 tokenA and 1 tokenB when the pair is in balance.
For example, DAI-USDC Pair https://etherscan.io/address/0xae461ca67b15dc8dc81ce7615e0320da1a9ab8d5 returns 18 in function `decimals()`.
On the other hand, the Well contract uses 18 decimals, but its decimals are, in fact, 24 because it uses `x * y * 1e12` as the token supply.

### Recommendation
Save `x * y * 1e12` in a new state variable `_tokenSupply` and override `tokenSupply()` to return truncated least significant 6 digits.


