## [01] Potential loss of tokens for first liquidity provider during zero LP minting  

In the dual token pool, the first liquidity provider may inadvertently lose tokens when they assign a value to one token and set the second token amount to zero. Notably, in the `_addLiquidity()` function on line https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L441 when the `_mint()` method is invoked, there is no validation check for the `lpAmountOut` variable to ensure it is not zero.

Please find below a Proof of Concept. Create a new file named like `Well.AddLiquidityAttack.t.sol` and include the following context.

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

import {console} from "forge-std/Test.sol";
import {TestHelper, IERC20, Call, Balances} from "test/TestHelper.sol";
import {AddLiquidityAction, RemoveLiquidityAction, LiquidityHelper} from "test/LiquidityHelper.sol";
import {Aquifer} from "src/Aquifer.sol";
import {SwapHelper, SwapAction, Snapshot} from "test/SwapHelper.sol";

contract WellAddLiquidityTest is LiquidityHelper {

    function _setupWellWithoutAddingLiquidity(Call memory _wellFunction, Call[] memory _pumps, IERC20[] memory _tokens) internal {
        tokens = _tokens;
        wellFunction = _wellFunction;
        for (uint256 i; i < _pumps.length; i++) {
            pumps.push(_pumps[i]);
        }

        initUser();

        wellImplementation = deployWellImplementation();
        aquifer = new Aquifer();
        well = encodeAndBoreWell(address(aquifer), wellImplementation, tokens, _wellFunction, _pumps, bytes32(0));

        // Mint mock tokens to user
        mintTokens(user, initialLiquidity);
        mintTokens(user2, initialLiquidity);
        approveMaxTokens(user, address(well));
        approveMaxTokens(user2, address(well));

        // Mint mock tokens to TestHelper
        mintTokens(address(this), initialLiquidity);
        approveMaxTokens(address(this), address(well));

        // Do not add initial liquidity from TestHelper
        // addLiquidityEqualAmount(address(this), initialLiquidity);
    }

    function setUp() public {
        _setupWellWithoutAddingLiquidity(deployWellFunction(), deployPumps(1), deployMockTokens(2));
    }

    function _addLiq(address targetUser, uint256 token0, uint256 token1) internal returns(uint256 minted) {
        uint256[] memory amounts = new uint256[](tokens.length);
        amounts[0] = token0;
        amounts[1] = token1;
        minted = well.addLiquidity(amounts, 0, targetUser, type(uint256).max); 
    }

    function testZeroMint() public {
        vm.startPrank(user);
        uint256 userBalanceToken0Before = IERC20(tokens[0]).balanceOf(address(user));
        _addLiq(user, 1e18, 0);
        uint256 userBalanceToken0After = IERC20(tokens[0]).balanceOf(address(user));
        assert(
            // Spent `1e18` token0
            userBalanceToken0Before - userBalanceToken0After == 1e18
            // Got zero LP tokens
            && well.balanceOf(user) == 0
            );
    }
}
```
Begin by executing `forge test -vv --match-test testZeroMint`. Here is an example output:
```bash
Running 1 test for test/Well.AddLiquidityAttack.t.sol:WellAddLiquidityTest
[PASS] testZeroMint() (gas: 138179)
Test result: ok. 1 passed; 0 failed; finished in 4.36ms
```

I advise that transactions be reverted when zero LP tokens are being minted.

## [02] Minting the first `MINIMUM_LIQUIDITY` tokens to the zero address

The practice of minting the first `MINIMUM_LIQUIDITY` tokens (1000 tokens in the case of Uniswap) to the zero `address(0x0)` when initializing a new liquidity pool is a design choice utilized by the Uniswap protocol. This mechanism is used to prevent anyone from ever owning a 100% share of the liquidity pool, which could potentially manipulate the price of tokens in the pool.

Implementing this feature can bring a number of benefits including increased robustness against potential pricing manipulation.

I recommend that the `_addLiquidity()` function at line https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L413 be updated to mint the first `MINIMUM_LIQUIDITY` tokens to the zero address, as is done in Uniswap.
