## QA Summary<a name="QA Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#low1-consider-the-case-where-totalsupply-is-0) | Consider the case where `lpTokenSupply` is 0 | 2 |
| [LOW&#x2011;2](#low2-draft-import-dependencies) | Draft Import Dependencies | 1 |
| [LOW&#x2011;3](#low3-function-can-risk-gas-exhaustion-on-large-receipt-calls-due-to-multiple-mandatory-loops) | Function can risk gas exhaustion on large receipt calls due to multiple mandatory loops | 1 |
| [LOW&#x2011;4](#low4-missing-contractexistence-checks-before-lowlevel-calls) | Missing Contract-existence Checks Before Low-level Calls | 1 |
| [LOW&#x2011;5](#low5-safetransfer-function-does-not-check-for-contract-existence) | `safeTransfer` function does not check for contract existence | 1 |
| [LOW&#x2011;6](#low6-solidity-version-0820-may-not-work-on-other-chains-due-to-push0) | Solidity version 0.8.20 may not work on other chains due to `PUSH0` | 0 |
| [LOW&#x2011;7](#low7-consider-case-where-the-variable-will-not-be-set-and-will-default-to-zero) | Consider case where the variable will not be set and will default to zero | 1 |

Total: 7 contexts over 7 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#nc1-consider-adding-a-denylist) | Consider adding a deny-list | 1 |
| [NC&#x2011;2](#nc2-no-need-to-initialize-uints-to-zero) | No need to initialize uints to zero | 1 |
| [NC&#x2011;3](#nc19-use-smtchecker) | Use SMTChecker | 1 |

Total: 3 contexts over 3 issues

## Low Risk Issues


### <a href="#qa-summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Consider the case where `lpTokenSupply` is 0

Consider the case where `lpTokenSupply` is 0. When `lpTokenSupply` is 0, it should return 0 directly, because there will be an error of dividing by 0.

#### <ins>Impact</ins>

This would cause the affected functions to revert and as a result can lead to potential loss.

#### <ins>Proof Of Concept</ins>

```solidity
File: ProportionalLPToken2.sol

22: underlyingAmounts[0] = lpTokenAmount * reserves[0] / lpTokenSupply;
23: underlyingAmounts[1] = lpTokenAmount * reserves[1] / lpTokenSupply;

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/functions/ProportionalLPToken2.sol#L22-L23




#### <ins>Recommended Mitigation Steps</ins>
Add check for zero value and return 0.

```solidity
if ( lpTokenSupply == 0) return 0;
```



### <a href="#qa-summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> Draft Import Dependencies
Contracts are importing draft dependencies. These imported contracts are still a draft and are not considered ready for mainnet use and have not received adequate security auditing or are liable to change with future development.

#### <ins>Proof Of Concept</ins>


```solidity
File: Well.sol

6: import {ERC20Upgradeable, ERC20PermitUpgradeable} from "ozu/token/ERC20/extensions/draft-ERC20PermitUpgradeable.sol";

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L6



#### <ins>Recommended Mitigation Steps</ins>

Stop using draft imported contracts and use their release version if it exists.




### <a href="#qa-summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Function can risk gas exhaustion on large receipt calls due to multiple mandatory loops

The function contains loops over arrays that the user cannot influence. 
In any call for the function, users spend gas to iterate over the array. There are loops in the following functions that iterate all array values. Which could risk gas exhaustion on large array calls.

This risk is small, but could be lessened somewhat by allowing separate calls for small loops.
Consider allowing partial calls via array arguments, or detecting expensive call bundles and only partially iterating over them. Since the logic already filters out calls, this would make the later loops smaller.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: MultiFlowPump.sol

285: function _readCumulativeReserves(address well) internal view returns (bytes16[] memory cumulativeReserves) {
        bytes32 slot = _getSlotForAddress(well);
        uint256[] memory reserves = IWell(well).getReserves();
        (uint8 numberOfReserves, uint40 lastTimestamp, bytes16[] memory lastReserves) = slot.readLastReserves();
        if (numberOfReserves == 0) {
            revert NotInitialized();
        }
        uint256 offset = _getSlotsOffset(numberOfReserves) << 1;
        assembly {
            slot := add(slot, offset)
        }
        cumulativeReserves = slot.readBytes16(numberOfReserves);
        uint256 deltaTimestamp = _getDeltaTimestamp(lastTimestamp);
        bytes16 deltaTimestampBytes = deltaTimestamp.fromUInt();
        bytes16 blocksPassed = (deltaTimestamp / BLOCK_TIME).fromUInt();
        
        for (uint256 i; i < cumulativeReserves.length; ++i) {
            lastReserves[i] = _capReserve(lastReserves[i], reserves[i].fromUIntToLog2(), blocksPassed);
            cumulativeReserves[i] = cumulativeReserves[i].add(lastReserves[i].mul(deltaTimestampBytes));
        }
    }

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L285



</details>






### <a href="#qa-summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> Missing Contract-existence Checks Before Low-level Calls

Low-level calls return success if there is no code present at the specified address. 

#### <ins>Proof Of Concept</ins>


```solidity
File: Aquifer.sol

55: (bool success, bytes memory returnData) = well.call(initFunctionCall);
```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Aquifer.sol#L55




#### <ins>Recommended Mitigation Steps</ins>

In addition to the zero-address checks, add a check to verify that `<address>.code.length > 0`




### <a href="#qa-summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> `safeTransfer` function does not check for contract existence

According to the implementation of the of the library's safeTransfer function, the contract existences are not checked. It is possible that a token becomes non-existent in the future; for instance, if some bugs are found for the output token, it is possible that the output token will be destroyed through calling its selfdestruct function after migrating its data to another contract for fixing these bugs. When this happens, such output token becomes non-existent but calling the swap function to swap for it does not revert. As a result, the user can lose tokens 

#### <ins>Proof Of Concept</ins>


```solidity
File: Well.sol

774: function _safeTransferFromFeeOnTransfer(
        IERC20 token,
        address from,
        uint256 amount
    ) internal returns (uint256 amountTransferred) {
        uint256 balanceBefore = token.balanceOf(address(this));
        token.safeTransferFrom(from, address(this), amount);
        amountTransferred = token.balanceOf(address(this)) - balanceBefore;
    }

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L774



#### <ins>Recommended Mitigation Steps</ins>

In the library's `safeTransfer` function, the existence of the contract for token should be checked before performing the low-level call. If such contract does not exist, calling this function should revert.



### <a href="#qa-summary">[LOW&#x2011;6]</a><a name="LOW&#x2011;6"> Solidity version 0.8.20 may not work on other chains due to `PUSH0`

The compiler for Solidity 0.8.20 switches the default target EVM version to [Shanghai](https://blog.soliditylang.org/2023/05/10/solidity-0.8.20-release-announcement/#important-note), which includes the new `PUSH0` op code. This op code may not yet be implemented on all L2s, so deployment on these chains will fail. To work around this issue, use an earlier [EVM](https://docs.soliditylang.org/en/v0.8.20/using-the-compiler.html?ref=zaryabs.com#setting-the-evm-version-to-target) [version](https://book.getfoundry.sh/reference/config/solidity-compiler#evm_version). While the project itself may or may not compile with 0.8.20, other projects with which it integrates, or which extend this project may, and those projects will have problems deploying these contracts/libraries.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: Aquifer.sol

pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Aquifer.sol#L3

```solidity
File: Well.sol

pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L3

```solidity
File: ConstantProduct2.sol

pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-07-basin/tree/main/src/functions/ConstantProduct2.sol#L3

```solidity
File: ProportionalLPToken2.sol

pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-07-basin/tree/main/src/functions/ProportionalLPToken2.sol#L3

```solidity
File: LibBytes.sol

pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibBytes.sol#L3

```solidity
File: LibBytes16.sol

pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibBytes16.sol#L3

```solidity
File: LibContractInfo.sol

pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibContractInfo.sol#L3

```solidity
File: LibLastReserveBytes.sol

pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibLastReserveBytes.sol#L3

```solidity
File: LibWellConstructor.sol

pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibWellConstructor.sol#L4

```solidity
File: MultiFlowPump.sol

pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L3



</details>





### <a href="#qa-summary">[LOW&#x2011;7]</a><a name="LOW&#x2011;7"> Consider case where the variable will not be set and will default to zero
As per the `_init` function in the `MultiFlowPump.sol` contract will just continue its logic even though `reserves` was not set (0 value) as there is no revert case when `reserves` is equal to 0

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: MultiFlowPump.sol

146: function _init(bytes32 slot, uint40 lastTimestamp, uint256[] memory reserves) internal {
        uint256 numberOfReserves = reserves.length;
        bytes16[] memory byteReserves = new bytes16[](numberOfReserves);

        // Skip {_capReserve} since we have no prior reference

        for (uint256 i; i < numberOfReserves; ++i) {
            if (reserves[i] == 0) return;
            byteReserves[i] = reserves[i].fromUIntToLog2();
        }

        // Write: Last Timestamp & Last Reserves
        slot.storeLastReserves(lastTimestamp, byteReserves);

        // Write: EMA Reserves
        // Start at the slot after `byteReserves`
        uint256 numSlots = _getSlotsOffset(byteReserves.length);
        assembly {
            slot := add(slot, numSlots)
        }
        slot.storeBytes16(byteReserves); // EMA Reserves
    }
```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L146


## Non Critical Issues


### <a href="#qa-summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Consider adding a deny-list

Doing so will significantly increase centralization, but will help to prevent hackers from using stolen tokens

#### <ins>Proof Of Concept</ins>


```solidity
File: Well.sol

22: contract Well is ERC20PermitUpgradeable, IWell, IWellErrors, ReentrancyGuardUpgradeable, ClonePlus

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L22











### <a href="#qa-summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> No need to initialize uints to zero

There is no need to initialize `uint` variables to zero as their default value is `0`

#### <ins>Proof Of Concept</ins>

```solidity
File: Well.sol

77: uint256 constant LOC_AQUIFER_ADDR = 0;

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L77







### <a href="#qa-summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19




