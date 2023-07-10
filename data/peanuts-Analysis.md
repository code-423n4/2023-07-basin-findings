## Table of Contents

1. Context
2. Approach
3. Mechanism Review
4. Architecture Recommendations
5. Systemic Risks

### 1. Context

Spent most of the time auditing the Well's code and briefly looked through the Math in the Pump contract. 

### 2. Approach

1. Read the two whitepapers and watch the video documentation first.
2. Try to understand how the Well and the Pump works together by looking at the high-level diagram given.
3. Look at each code individually and focus on the internal function calls.
4. Read the test files to get a better idea of the end-to-end scenarios.

### 3. Mechanism Review

1. The addition of both 'swapExactInputSingle' (swapFrom) and 'swapExactOutputSingle' (swapTo) style when swapping the tokens is appreciated and well created, especially since the latter is not very common. 

2. `SwapTo()` calculates the input amount before transferring the tokens in. This is unlike how Uniswap does it, which is transferring the maximum amount of tokens in and then refunding the remaining tokens back. The protocol does it better and saves some gas in the process as well by doing just one transfer. 

3. Shift is an interesting concept as it opts away from using the router and let the Wells interact with each other instead. Saves some gas by not transferring tokens around many times. 

4. Deadline, slippage is set properly in every swap and increase/decrease liquidity. 

5. Getter functions are well written and plenty; users have access to many external view functions to get a sense of what they are doing before calling the actual function.

### 4. Architecture Recommendations

1. I recommend that the swap functions (`swapFrom()` and `swapFromFeeOnTransfer()`) should have a ` bool feeOnTransfer` flag the same as `addLiquidity()` in order to save gas in case the user accidentally calls the wrong function. This saves the user some gas in case he wants to use fee-on-transfer token but calls `swapFrom()` instead, and then has to wait until `_setReserves()` is calculated before the function reverts, which costs quite a bit of gas for the computation. Should check whether the swap uses fee-on-transfer tokens from the very start as such:

```
    function swapFrom(
        IERC20 fromToken,
        IERC20 toToken,
        uint256 amountIn,
        uint256 minAmountOut,
        address recipient,
        uint256 deadline,
+       bool isFeeOnTransfer
    ) external nonReentrant expire(deadline) returns (uint256 amountOut) {
+       if(isFeeOnTransfer) revert();
        fromToken.safeTransferFrom(msg.sender, address(this), amountIn);
```

2. The swapTo function can also include the feeOnTransfer parameter (must always be false) to save some gas in case the user unknowningly uses the `swapTo()` function with a fee-on-transfer token.

3. Since the Well function is awaiting a proper implementation, protocol could give an example of how to edit the Well contract. For example, when updating pump, the protocol can give a possible commented code block if an external shutoff mechanism were to be added:

```
            try IPump(_pump.target).update(reserves, _pump.data) {}
            catch {
                // ignore reversion. If an external shutoff mechanism is added to a Pump, it could be called here.
+           // Example of how a shutoff mechanism would look like in code
 }
```

### 5. Systemic Risks

Since the creation of the Well clone is permissionless, the implementation contract has to be airtight to make sure that the creators of the well cannot exploit the well through inflation attacks or any sort of attacks when minting LPs, swapping or decreasing liquidity. This poses a huge systemic risk because if the implementation fails, every clone that clones through the foiled implementation will also be at risk.

Also, even though the Wells are permissionless, if any Well user is malicious (if there is an opportunity to be malicious), then it will affect the reputation of the protocol since the whole system relies on the Well contracts.


### Time spent:
15 hours