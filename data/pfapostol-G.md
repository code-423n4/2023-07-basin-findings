## Gas report

Foundry config setup:

```diff
diff --git a/foundry.toml b/foundry.toml
index f4080ba..3df885b 100644
--- a/foundry.toml
+++ b/foundry.toml
@@ -10,6 +10,7 @@ remappings = [
 ]
 block_number = 16826654
 block_timestamp = 1678803642
+gas_reports = ['ConstantProduct2', 'ProportionalLPToken2', 'MultiFlowPump', 'Aquifer', 'Well', 'LibBytes', 'LibBytes16', 'LibContractInfo', 'LibLastReserveBytes', 'LibWellConstructor']

 [rpc_endpoints]
 mainnet = "${MAINNET_RPC_URL}"
 ```

- [Gas report](#gas-report)
- [Public function for the public varaible](#public-function-for-the-public-varaible)
    - [Code difference](#code-difference)
- [Refactoring](#refactoring)
  - [Return early as soon as the indices have been found to prevent unnecessary looping through the array.](#return-early-as-soon-as-the-indices-have-been-found-to-prevent-unnecessary-looping-through-the-array)
    - [Code difference](#code-difference-1)
  - [Don't write state changes until slippage has been checked for all tokens](#dont-write-state-changes-until-slippage-has-been-checked-for-all-tokens)
    - [Code difference](#code-difference-2)
  - [In `removeLiquidityImbalanced` check slippage before transfering tokens](#in-removeliquidityimbalanced-check-slippage-before-transfering-tokens)
    - [Code difference](#code-difference-3)
  - [`MultiFlowPump` check for reserves != 0 twice](#multiflowpump-check-for-reserves--0-twice)
    - [Code difference](#code-difference-4)

## Public function for the public varaible
<a name="G-01"></a> 
The external function `wellImplementation` only calls the public state variable `wellImplementations`, which can be called directly without creating additional functions.

- [src\Aquifer.sol](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Aquifer.sol#L86)


`forge test --ffi --gas-report --match-contract AquiferTest`

Deployment gas cost reduced by **600**

Minimum functions call gas cost reduced by **9**

Average functions call gas cost reduced by **9**

Maximum functions call gas cost reduced by **9**

#### Code difference

```diff
diff --git a/src/Aquifer.sol b/src/Aquifer.sol
index 7bbc539..f768676 100644
--- a/src/Aquifer.sol
+++ b/src/Aquifer.sol
@@ -22,7 +22,7 @@ contract Aquifer is IAquifer, ReentrancyGuard {

    // A mapping of Well address to the Well implementation addresses
    // Mapping gets set on Well deployment
-    mapping(address => address) wellImplementations;
+    mapping(address => address) public wellImplementations;

    constructor() ReentrancyGuard() {}

@@ -82,8 +82,4 @@ contract Aquifer is IAquifer, ReentrancyGuard {
            IWell(well).wellData()
        );
    }
-
-    function wellImplementation(address well) external view returns (address implementation) {
-        return wellImplementations[well];
-    }
}
```

## Refactoring

### Return early as soon as the indices have been found to prevent unnecessary looping through the array.

- [src\Well.sol](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L730)

#### Code difference

```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..6035f14 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -743,10 +743,12 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
                j = k;
                foundJ = true;
            }
+            if (foundI && foundJ) {
+                return (i, j);
+            }
        }

-        if (!foundI) revert InvalidTokens();
-        if (!foundJ) revert InvalidTokens();
+        revert InvalidTokens();
    }

    /**
```

`_getIJ` is an internal function so forge doesn't generate gas report for `getIJ` tests so I use direct output

The number of tokens and their position is not constant, so the following test is just an example, confirming that when scaling, the savings in the best case (when indices are close to the beginning of the array) outweigh the losses in the worst case (indices at the end of the array). An average result of 5000 runs with random indexes is always more efficient for new code.

Command: `forge test --gas-report --ffi --match-path "test/Well.IJ.t.sol" --fuzz-runs 5000`

<details>
    <summary>OLD code perfomance</summary>

    ```
    [PASS] test_best_case() (gas: 380721)
    [PASS] test_search(uint256,uint256) (runs: 5000, μ: 390201, ~: 390457)
    [PASS] test_worst_case() (gas: 380745)
    ```
</details>

<details>
    <summary>NEW code perfomance</summary>

    ```
    [PASS] test_best_case() (gas: 332253)
    [PASS] test_search(uint256,uint256) (runs: 5000, μ: 380115, ~: 384689)
    [PASS] test_worst_case() (gas: 386662)
    ```
</details>

<details>
    <summary>test/Well.IJ.t.sol code</summary>

    ```solidity
    // SPDX-License-Identifier: MIT

    pragma solidity ^0.8.17;

    import {Test, console} from "forge-std/Test.sol";
    import {Well, IERC20} from "test/TestHelper.sol";

    contract WellSearchTest is Well, Test {
        IERC20[] public _tokens;

        function setUp() public {
            for (uint256 i = 0; i <= 150; ++i) {
                _tokens.push(IERC20(address(bytes20(bytes32(generatePrivateKey())))));
            }
        }

        function test_search(uint256 i, uint256 j) public {
            i = bound(i, 0, 150);
            j = bound(j, 0, 150);
            vm.assume(i != j);

            IERC20 i_token = _tokens[i];
            IERC20 j_token = _tokens[j];

            (uint256 found_i, uint256 found_j) = _getIJ(_tokens, i_token, j_token);

            assert(found_i == i);
            assert(found_j == j);
        }

        function test_best_case() public {
            IERC20 i_token = _tokens[0];
            IERC20 j_token = _tokens[1];

            (uint256 found_i, uint256 found_j) = _getIJ(_tokens, i_token, j_token);

            assert(found_i == 0);
            assert(found_j == 1);
        }

        function test_worst_case() public {
            IERC20 i_token = _tokens[149];
            IERC20 j_token = _tokens[150];

            (uint256 found_i, uint256 found_j) = _getIJ(_tokens, i_token, j_token);

            assert(found_i == 149);
            assert(found_j == 150);
        }

        function generatePrivateKey() public returns (bytes memory) {
            bytes memory privKeyData;

            string[] memory inputs = new string[](3);
            inputs[0] = "cast";
            inputs[1] = "wallet";
            inputs[2] = "new";

            // execute `cast wallet new` and return the output to res
            bytes memory res = vm.ffi(inputs);

            // cast private key is returned in last 64 bytes at the end of res
            privKeyData = slice(res, res.length - 64, 64);

            // convert the weird ASCII-byte values returned by cast call into a valid bytes32 value
            return privKeyData;
        }

        function slice(
            bytes memory _bytes,
            uint256 _start,
            uint256 _length
        )
            internal
            pure
            returns (bytes memory)
        {
            require(_length + 31 >= _length, "slice_overflow");
            require(_bytes.length >= _start + _length, "slice_outOfBounds");

            bytes memory tempBytes;

            assembly {
                switch iszero(_length)
                case 0 {
                    // Get a location of some free memory and store it in tempBytes as
                    // Solidity does for memory variables.
                    tempBytes := mload(0x40)

                    // The first word of the slice result is potentially a partial
                    // word read from the original array. To read it, we calculate
                    // the length of that partial word and start copying that many
                    // bytes into the array. The first word we copy will start with
                    // data we don't care about, but the last `lengthmod` bytes will
                    // land at the beginning of the contents of the new array. When
                    // we're done copying, we overwrite the full first word with
                    // the actual length of the slice.
                    let lengthmod := and(_length, 31)

                    // The multiplication in the next line is necessary
                    // because when slicing multiples of 32 bytes (lengthmod == 0)
                    // the following copy loop was copying the origin's length
                    // and then ending prematurely not copying everything it should.
                    let mc := add(add(tempBytes, lengthmod), mul(0x20, iszero(lengthmod)))
                    let end := add(mc, _length)

                    for {
                        // The multiplication in the next line has the same exact purpose
                        // as the one above.
                        let cc := add(add(add(_bytes, lengthmod), mul(0x20, iszero(lengthmod))), _start)
                    } lt(mc, end) {
                        mc := add(mc, 0x20)
                        cc := add(cc, 0x20)
                    } {
                        mstore(mc, mload(cc))
                    }

                    mstore(tempBytes, _length)

                    //update free-memory pointer
                    //allocating the array padded to 32 bytes like the compiler does now
                    mstore(0x40, and(add(mc, 31), not(31)))
                }
                //if we want a zero-length slice let's just return a zero-length array
                default {
                    tempBytes := mload(0x40)
                    //zero out the 32 bytes slice we are about to return
                    //we need to do it because Solidity does not garbage collect
                    mstore(tempBytes, 0)

                    mstore(0x40, add(tempBytes, 0x20))
                }
            }

            return tempBytes;
        }
    }
    ```

</details>

### Don't write state changes until slippage has been checked for all tokens

- [src\Well.sol](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L474)

Check the slippage of each token before transferring, as the transfer causes a state change, and a subsequent revert will revert the changes to the initial state. By refactoring the code, you can save ≈30000+ gas per each token before the token that triggers the revert.

#### Code difference

```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..ef6f9fc 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -474,6 +474,8 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
             if (tokenAmountsOut[i] < minTokenAmountsOut[i]) {
                 revert SlippageOut(tokenAmountsOut[i], minTokenAmountsOut[i]);
             }
+        }
+        for (uint256 i; i < _tokens.length; ++i) {
             _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);
             reserves[i] = reserves[i] - tokenAmountsOut[i];
         }
```

Command: `forge test --gas-report --ffi --match-path "test/Well.Splippage.t.sol"`

Deployment gas cost increased by **5 808**

Average functions call gas cost reduced by **11 287**

Maximum functions call gas cost increased by **253**



<details>
    <summary>OLD code perfomance:</summary>

    ```
    [PASS] test_removeLiquidity() (gas: 175816)
    [PASS] test_removeLiquidity_revertIf_SlippageOut_first() (gas: 53940)
    [PASS] test_removeLiquidity_revertIf_SlippageOut_second() (gas: 88262)

    | src/Well.sol:Well contract |                 |       |        |       |         |
    | -------------------------- | --------------- | ----- | ------ | ----- | ------- |
    | Deployment Cost            | Deployment Size |       |        |       |         |
    | 4036006                    | 20189           |       |        |       |         |
    | Function Name              | min             | avg   | median | max   | # calls |
    | ...                        | ...             | ...   | ...    | ...   | ...     |
    | removeLiquidity            | 33567           | 62846 | 67911  | 87060 | 3       |
    ```
</details>

<details>
    <summary>NEW code perfomance:</summary>

    ```
    [PASS] test_removeLiquidity() (gas: 176069)
    [PASS] test_removeLiquidity_revertIf_SlippageOut_first() (gas: 53940)
    [PASS] test_removeLiquidity_revertIf_SlippageOut_second() (gas: 54150)
    | src/Well.sol:Well contract |                 |       |        |       |         |
    | -------------------------- | --------------- | ----- | ------ | ----- | ------- |
    | Deployment Cost            | Deployment Size |       |        |       |         |
    | 4041814                    | 20218           |       |        |       |         |
    | Function Name              | min             | avg   | median | max   | # calls |
    | ...                        | ...             | ...   | ...    | ...   | ...     |
    | removeLiquidity            | 33567           | 51559 | 33799  | 87313 | 3       |
    ```
</details>


<details>
    <summary>test/Well.Splippage.t.sol</summary>

    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.17;

    import {TestHelper, ConstantProduct2, IERC20, Balances} from "test/TestHelper.sol";
    import {Snapshot, AddLiquidityAction, RemoveLiquidityAction, LiquidityHelper} from "test/LiquidityHelper.sol";
    import {IWell} from "src/interfaces/IWell.sol";
    import {IWellErrors} from "src/interfaces/IWellErrors.sol";

    contract WellSlippageOutTest is LiquidityHelper {
        ConstantProduct2 cp;
        bytes constant data = "";
        uint256 constant addedLiquidity = 1000 * 1e18;

        function setUp() public {
            cp = new ConstantProduct2();
            setupWell(2);

            // Add liquidity. `user` now has (2 * 1000 * 1e27) LP tokens
            addLiquidityEqualAmount(user, addedLiquidity);
        }

        /// @dev removeLiquidity: remove to equal amounts of underlying
        function test_removeLiquidity() public prank(user) {
            uint256 lpAmountIn = 1000 * 1e24;

            uint256[] memory amountsOut = new uint256[](2);
            amountsOut[0] = 1000 * 1e18;
            amountsOut[1] = 1000 * 1e18;

            Snapshot memory before;
            RemoveLiquidityAction memory action;

            action.amounts = amountsOut;
            action.lpAmountIn = lpAmountIn;
            action.recipient = user;
            action.fees = new uint256[](2);

            (before, action) = beforeRemoveLiquidity(action);
            well.removeLiquidity(lpAmountIn, amountsOut, user, type(uint256).max);
            afterRemoveLiquidity(before, action);
            checkInvariant(address(well));
        }

        /// @dev removeLiquidity: reverts when user tries to remove too much of an underlying token
        function test_removeLiquidity_revertIf_SlippageOut_first() public prank(user) {
            uint256 lpAmountIn = 1000 * 1e24;

            uint256[] memory minTokenAmountsOut = new uint256[](2);
            minTokenAmountsOut[0] = 1001 * 1e18; // too high
            minTokenAmountsOut[1] = 1000 * 1e18;

            vm.expectRevert(abi.encodeWithSelector(IWellErrors.SlippageOut.selector, 1000 * 1e18, minTokenAmountsOut[0]));
            well.removeLiquidity(lpAmountIn, minTokenAmountsOut, user, type(uint256).max);
        }

        function test_removeLiquidity_revertIf_SlippageOut_second() public prank(user) {
            uint256 lpAmountIn = 1000 * 1e24;

            uint256[] memory minTokenAmountsOut = new uint256[](2);
            minTokenAmountsOut[0] = 1000 * 1e18;
            minTokenAmountsOut[1] = 1001 * 1e18; // too high

            vm.expectRevert(abi.encodeWithSelector(IWellErrors.SlippageOut.selector, 1000 * 1e18, minTokenAmountsOut[1]));
            well.removeLiquidity(lpAmountIn, minTokenAmountsOut, user, type(uint256).max);
        }

    }
    ```
</details>

### In `removeLiquidityImbalanced` check slippage before transfering tokens

- [src\Well.sol](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L558)

**NOTE**: `_calcLpTokenSupply` should not use token balances, which is not provided by the interface anyway, but the user can do it manually.

#### Code difference

```diff
diff --git a/src/Well.sol b/src/Well.sol
index 64dd3cc..f71371e 100644
--- a/src/Well.sol
+++ b/src/Well.sol
@@ -555,7 +555,6 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
         uint256[] memory reserves = _updatePumps(_tokens.length);

         for (uint256 i; i < _tokens.length; ++i) {
-            _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);
             reserves[i] = reserves[i] - tokenAmountsOut[i];
         }

@@ -563,6 +562,9 @@ contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgr
         if (lpAmountIn > maxLpAmountIn) {
             revert SlippageIn(lpAmountIn, maxLpAmountIn);
         }
+        for (uint256 i; i < _tokens.length; ++i) {
+            _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);
+        }
         _burn(msg.sender, lpAmountIn);

         _setReserves(_tokens, reserves);
```

`forge test --gas-report --ffi --match-path "test/Well.SlippageIn.t.sol"`

Deployment gas cost increased by **5 808**

Minimum functions call gas cost reduced by **67 736**

Average functions call gas cost reduced by **33 742**

Maximum functions call gas cost increased by **253**


<details>
    <summary>OLD code perfomance:</summary>

    
    ```
    [PASS] test_removeLiquidityImbalanced() (gas: 172798)
    [PASS] test_removeLiquidityImbalanced_revertIf_notEnoughLP() (gas: 118619)
    | src/Well.sol:Well contract |                 |       |        |       |         |
    | -------------------------- | --------------- | ----- | ------ | ----- | ------- |
    | Deployment Cost            | Deployment Size |       |        |       |         |
    | 4036006                    | 20189           |       |        |       |         |
    | Function Name              | min             | avg   | median | max   | # calls |
    | ...                        | ...             | ...   | ...    | ...   | ...     |
    | removeLiquidityImbalanced  | 95369           | 96679 | 96679  | 97989 | 2       |
    ```
</details>

<details>
    <summary>NEW code perfomance:</summary>

    ```
    [PASS] test_removeLiquidityImbalanced() (gas: 173051)
    [PASS] test_removeLiquidityImbalanced_revertIf_notEnoughLP() (gas: 50883)
    | src/Well.sol:Well contract |                 |       |        |       |         |
    | -------------------------- | --------------- | ----- | ------ | ----- | ------- |
    | Deployment Cost            | Deployment Size |       |        |       |         |
    | 4041814                    | 20218           |       |        |       |         |
    | Function Name              | min             | avg   | median | max   | # calls |
    | ...                        | ...             | ...   | ...    | ...   | ...     |
    | removeLiquidityImbalanced  | 27633           | 62937 | 62937  | 98242 | 2       |
    ```
</details>

<details>
    <summary>test/Well.SlippageIn.t.sol</summary>

    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.17;

    import {TestHelper, ConstantProduct2, Balances} from "test/TestHelper.sol";
    import {IWell} from "src/interfaces/IWell.sol";
    import {IWellErrors} from "src/interfaces/IWellErrors.sol";

    contract WellRemoveLiquidityImbalancedTest is TestHelper {
        event RemoveLiquidity(uint256 lpAmountIn, uint256[] tokenAmountsOut, address recipient);

        uint256[] tokenAmountsOut;
        uint256 requiredLpAmountIn;

        // Setup
        ConstantProduct2 cp;
        uint256 constant addedLiquidity = 1000 * 1e18;

        function setUp() public {
            cp = new ConstantProduct2();
            setupWell(2);

            // Add liquidity. `user` now has (2 * 1000 * 1e27) LP tokens
            addLiquidityEqualAmount(user, addedLiquidity);

            // Shared removal amounts
            tokenAmountsOut.push(500 * 1e18); // 500   token0
            tokenAmountsOut.push(506 * 1e17); //  50.6 token1
            requiredLpAmountIn = 290 * 1e24; // LP needed to remove `tokenAmountsOut`
        }

        /// @dev Base case
        function test_removeLiquidityImbalanced() public prank(user) {
            Balances memory userBalanceBefore = getBalances(user, well);

            uint256 initialLpAmount = userBalanceBefore.lp;
            uint256 maxLpAmountIn = requiredLpAmountIn;

            vm.expectEmit(true, true, true, true);
            emit RemoveLiquidity(maxLpAmountIn, tokenAmountsOut, user);
            well.removeLiquidityImbalanced(maxLpAmountIn, tokenAmountsOut, user, type(uint256).max);

            Balances memory userBalanceAfter = getBalances(user, well);
            Balances memory wellBalanceAfter = getBalances(address(well), well);

            // `user` balance of LP tokens decreases
            assertEq(userBalanceAfter.lp, initialLpAmount - maxLpAmountIn);

            // `user` balance of underlying tokens increases
            // assumes initial balance of zero
            assertEq(userBalanceAfter.tokens[0], tokenAmountsOut[0], "Incorrect token0 user balance");
            assertEq(userBalanceAfter.tokens[1], tokenAmountsOut[1], "Incorrect token1 user balance");

            // Well's reserve of underlying tokens decreases
            assertEq(wellBalanceAfter.tokens[0], 1500 * 1e18, "Incorrect token0 well reserve");
            assertEq(wellBalanceAfter.tokens[1], 19_494 * 1e17, "Incorrect token1 well reserve");
            checkInvariant(address(well));
        }

        function test_removeLiquidityImbalanced_revertIf_notEnoughLP() public prank(user) {
            uint256 maxLpAmountIn = 5 * 1e24;
            vm.expectRevert(abi.encodeWithSelector(IWellErrors.SlippageIn.selector, requiredLpAmountIn, maxLpAmountIn));
            well.removeLiquidityImbalanced(maxLpAmountIn, tokenAmountsOut, user, type(uint256).max);
        }
    }
    ```
</details>



### `MultiFlowPump` check for reserves != 0 twice

- [src\pumps\MultiFlowPump.sol](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/pumps/MultiFlowPump.sol#L86)

#### Code difference

```diff
diff --git a/src/pumps/MultiFlowPump.sol b/src/pumps/MultiFlowPump.sol
index e2aab5d..a3ee999 100644
--- a/src/pumps/MultiFlowPump.sol
+++ b/src/pumps/MultiFlowPump.sol
@@ -81,10 +81,6 @@ contract MultiFlowPump is IPump, IMultiFlowPumpErrors, IInstantaneousPump, ICumu

         // If the last timestamp is 0, then the pump has never been used before.
         if (pumpState.lastTimestamp == 0) {
-            for (uint256 i; i < numberOfReserves; ++i) {
-                // If a reserve is 0, then the pump cannot be initialized.
-                if (reserves[i] == 0) return;
-            }
             _init(slot, uint40(block.timestamp), reserves);
             return;
         }
```

`forge test --gas-report --ffi --match-path "test/pumps/Pump.Update.Init.t.sol"`


Deployment gas cost reduced by **13 816**

Minimum functions call gas cost increased by **356**

Average functions call gas cost reduced by **15**

Maximum functions call gas cost reduced by **386**


<details>
    <summary>OLD code perfomance:</summary>

    ```
    [PASS] test_first_update_init() (gas: 195951)
    | src/pumps/MultiFlowPump.sol:MultiFlowPump contract |                 |       |        |       |         |
    |----------------------------------------------------|-----------------|-------|--------|-------|---------|
    | Deployment Cost                                    | Deployment Size |       |        |       |         |
    | 3325658                                            | 18861           |       |        |       |         |
    | Function Name                                      | min             | avg   | median | max   | # calls |
    | update                                             | 3157            | 40385 | 40385  | 77614 | 2       |
    ```
</details>

<details>
    <summary>NEW code perfomance:</summary>

    ```
    [PASS] test_first_update_init() (gas: 195921)
    | src/pumps/MultiFlowPump.sol:MultiFlowPump contract |                 |       |        |       |         |
    |----------------------------------------------------|-----------------|-------|--------|-------|---------|
    | Deployment Cost                                    | Deployment Size |       |        |       |         |
    | 3311842                                            | 18792           |       |        |       |         |
    | Function Name                                      | min             | avg   | median | max   | # calls |
    | update                                             | 3513            | 40370 | 40370  | 77228 | 2       |
    ```
</details>

<details>
    <summary>test/pumps/Pump.Update.Init.t.sol</summary>

    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.17;

    import {console, TestHelper} from "test/TestHelper.sol";
    import {MultiFlowPump, ABDKMathQuad} from "src/pumps/MultiFlowPump.sol";
    import {from18, to18} from "test/pumps/PumpHelpers.sol";
    import {MockReserveWell} from "mocks/wells/MockReserveWell.sol";
    import {IMultiFlowPumpErrors} from "src/interfaces/pumps/IMultiFlowPumpErrors.sol";

    import {log2, powu, UD60x18, wrap, unwrap} from "prb/math/UD60x18.sol";
    import {exp2, log2, powu, UD60x18, wrap, unwrap, uUNIT} from "prb/math/UD60x18.sol";

    contract PumpUpdateTest is TestHelper {
        using ABDKMathQuad for bytes16;

        MultiFlowPump pump;
        MockReserveWell mWell;
        uint256[] b = new uint256[](2);

        uint256 constant BLOCK_TIME = 12;

        /// @dev for this test, `user` = a Well that's calling the Pump
        function setUp() public {
            mWell = new MockReserveWell();
            initUser();
            pump = new MultiFlowPump(
                from18(0.5e18), // cap reserves if changed +/- 50% per block
                from18(0.5e18), // cap reserves if changed +/- 50% per block
                12, // block time
                from18(0.9e18) // ema alpha
            );
        }

        function test_first_update_init() public {
            // Send first update to the Pump, which will initialize it
            vm.prank(user);
            b[0] = 1e6;
            b[1] = 2e6;
            mWell.update(address(pump), b, new bytes(0));
            mWell.update(address(pump), b, new bytes(0));
        }
    }

    ```
</details>
