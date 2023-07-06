**functions/ConstantProduct2.sol**
- L87/99 - You should check that ratios[i] is != 0, since that way an unhandled expectation would not be generated.


**functions/ProportionalLPToken2.sol**
- L22/23 - It should be checked that lpTokenSupply is != 0, since that way an unhandled expectation would not be generated.


**pumps/MultiFlowPump.sol**
- The order of the functions is not respected, external, public, internal.


**Aquifer.sol**
- L8/10 - Multiple contracts are imported that are never used, therefore adding them generates extra gas expense in the deploy and also reduces the understanding of the contract when the code is viewed in the blockchain explorer.


**Well.sol**
- L45-75 - The documentation is not written as documentation, it does not use naspect.

- L26-29/77-82 - The constants should all be together, to follow the solidity style rules. In addition to improving the understanding of the code.

- L120 - That the contract is called Well and that the well() function returns the status of various functions is a bit of a message, perhaps thinking of a better name would be a better decision, since it reduces the level of understanding of the name. A better one might be getState() or getWellStorage().

- L452/453/473/474/557/558/559/579/580 - The token array is traversed, without previously validating that the tokenAmountsIn arrays have the same length, this is important because it would generate an exception without being handled or it would be would execute without using all available tokenAmountsIn .

- L203/417/425/774 - The _safeTransferFromFeeOnTransfer() function mentions in its name that there is a fee for transfers, but in the implementation there is no such logic, therefore the name of the function is wrong, this can generate confusion to contract users. This also renders the bool feeOnTransfer input of _addLiquidity() useless. The same thing happens with the swapFromFeeOnTransfer() function.

- L730/759 - The _getIJ() function is not really necessary, since with _getJ() you could call it twice with a different J and get the same result. Therefore, it could be factored that way.
