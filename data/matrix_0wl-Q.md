## Non Critical Issues

|      | Issue                                                                                    |
| ---- | :--------------------------------------------------------------------------------------- |
| NC-1 | ADD TO INDEXED PARAMETER FOR COUNTABLE EVENTS                                            |
| NC-2 | ADD A TIMELOCK TO CRITICAL FUNCTIONS                                                     |
| NC-3 | GENERATE PERFECT CODE HEADERS EVERY TIME                                                 |
| NC-4 | CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT |
| NC-5 | DON’T WORRY ABOUT KECCAK256 COLLISIONS                                                   |
| NC-6 | FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION                            |
| NC-7 | MISSING EVENT FOR CRITICAL PARAMETER CHANGE                                              |
| NC-8 | Omissions in events                                                                      |
| NC-9 | USE A MORE RECENT VERSION OF SOLIDITY                                                    |

### [NC-1] ADD TO INDEXED PARAMETER FOR COUNTABLE EVENTS

#### Description:

Add to indexed parameter for countable Events

#### **Proof Of Concept**

```solidity
File: Aquifer.sol

76:         emit BoreWell(

```

```solidity
File: Well.sol

238:         emit Swap(fromToken, toToken, amountIn, amountOut, recipient);

305:         emit Swap(fromToken, toToken, amountIn, amountOut, recipient);

373:             emit Shift(reserves, tokenOut, amountOut, recipient);

443:         emit AddLiquidity(tokenAmountsIn, lpAmountOut, recipient);

482:         emit RemoveLiquidity(lpAmountIn, tokenAmountsOut, recipient);

516:         emit RemoveLiquidityOneToken(lpAmountIn, tokenOut, tokenAmountOut, recipient);

569:         emit RemoveLiquidity(lpAmountIn, tokenAmountsOut, recipient);

597:         emit Sync(reserves);

```

### [NC-2] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: Well.sol

239:         _setReserves(_tokens, reserves);

289:         _setReserves(_tokens, reserves);

372:             _setReserves(_tokens, reserves);

442:         _setReserves(_tokens, reserves);

481:         _setReserves(_tokens, reserves);

515:         _setReserves(_tokens, reserves);

568:         _setReserves(_tokens, reserves);

596:         _setReserves(_tokens, reserves);

632:     function _setReserves(IERC20[] memory _tokens, uint256[] memory reserves) internal {

```

### [NC-3] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

#### **Proof Of Concept**

```solidity
File: Aquifer.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: Well.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: functions/ConstantProduct2.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: functions/ProportionalLPToken2.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: libraries/LibBytes.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: libraries/LibBytes16.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: libraries/LibContractInfo.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: libraries/LibLastReserveBytes.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: libraries/LibWellConstructor.sol

4: pragma solidity ^0.8.17;

```

```solidity
File: pumps/MultiFlowPump.sol

3: pragma solidity ^0.8.17;

```

### [NC-4] CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

#### Description:

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.

WConstants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

#### **Proof Of Concept**

```solidity
File: Well.sol

29:     bytes32 constant RESERVES_STORAGE_SLOT = bytes32(uint256(keccak256("reserves.storage.slot")) - 1);

```

### [NC-5] DON’T WORRY ABOUT KECCAK256 COLLISIONS

#### Description:

When cancelBid is called, the commitment is set to 0 in order to force to fail the computeCommitment comparison.In case of the result of computeCommitment returns 0, because a collision or an error, a cancelled bid will be processed as valid.It’s more resilient to use a different value as flag, b.pubKey will be safer and cheaper.

#### **Proof Of Concept**

```solidity
File: Well.sol

424:                 if (tokenAmountsIn[i] == 0) continue;

430:                 if (tokenAmountsIn[i] == 0) continue;

```

### [NC-6] FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION

#### **Proof Of Concept**

```solidity
File: Aquifer.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: Well.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: functions/ConstantProduct2.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: functions/ProportionalLPToken2.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: libraries/LibBytes.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: libraries/LibBytes16.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: libraries/LibContractInfo.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: libraries/LibLastReserveBytes.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: libraries/LibWellConstructor.sol

4: pragma solidity ^0.8.17;

```

```solidity
File: pumps/MultiFlowPump.sol

3: pragma solidity ^0.8.17;

```

### [NC-7] MISSING EVENT FOR CRITICAL PARAMETER CHANGE

#### Description:

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

#### **Proof Of Concept**

```solidity
File: Aquifer.sol

76:         emit BoreWell(

```

```solidity
File: Well.sol

238:         emit Swap(fromToken, toToken, amountIn, amountOut, recipient);

305:         emit Swap(fromToken, toToken, amountIn, amountOut, recipient);

373:             emit Shift(reserves, tokenOut, amountOut, recipient);

443:         emit AddLiquidity(tokenAmountsIn, lpAmountOut, recipient);

482:         emit RemoveLiquidity(lpAmountIn, tokenAmountsOut, recipient);

516:         emit RemoveLiquidityOneToken(lpAmountIn, tokenOut, tokenAmountOut, recipient);

569:         emit RemoveLiquidity(lpAmountIn, tokenAmountsOut, recipient);

597:         emit Sync(reserves);

```

### [NC-8] Omissions in events

#### Description:

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible:

#### **Proof Of Concept**

```solidity
File: Well.sol

597:         emit Sync(reserves);

```

### [NC-9] USE A MORE RECENT VERSION OF SOLIDITY

#### Description:

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Proof Of Concept**

```solidity
File: Aquifer.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: Well.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: functions/ConstantProduct2.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: functions/ProportionalLPToken2.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: libraries/LibBytes.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: libraries/LibBytes16.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: libraries/LibContractInfo.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: libraries/LibLastReserveBytes.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: libraries/LibWellConstructor.sol

4: pragma solidity ^0.8.17;

```

```solidity
File: pumps/MultiFlowPump.sol

3: pragma solidity ^0.8.17;

```

## Low Issues

|     | Issue                                                                                             |
| --- | :------------------------------------------------------------------------------------------------ |
| L-1 | CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE or INITIAL VALUE CHECK IS MISSING IN SET FUNCTIONS |
| L-2 | DECODING AN IPFS HASH USING A FIXED HASH FUNCTION AND LENGTH OF THE HASH                          |
| L-3 | DRAFT OPENZEPPELIN DEPENDENCIES                                                                   |
| L-4 | LOSS OF PRECISION DUE TO ROUNDING                                                                 |
| L-5 | STACK TOO DEEP WHEN COMPILING                                                                     |
| L-6 | Timestamp Dependence                                                                              |
| L-7 | Account existence check for low-level calls                                                       |
| L-8 | USE `_SAFEMINT` INSTEAD OF `_MINT`                                                                |

### [L-1] CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE or INITIAL VALUE CHECK IS MISSING IN SET FUNCTIONS

#### Description:

The critical procedures should be two step process.

#### **Proof Of Concept**

```solidity
File: Well.sol

632:     function _setReserves(IERC20[] memory _tokens, uint256[] memory reserves) internal {

```

#### Recommended Mitigation Steps:

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

### [L-2] DECODING AN IPFS HASH USING A FIXED HASH FUNCTION AND LENGTH OF THE HASH

#### **Proof Of Concept**

```solidity
File: Aquifer.sol

60:                     returnData := add(returnData, 0x04)

```

```solidity
File: libraries/LibContractInfo.sol

23:                 mstore(add(symbol, 0x20), shl(224, shr(128, _contract)))

41:                 mstore(add(name, 0x20), shl(224, shr(128, _contract)))

```

#### Recommended Mitigation Steps:

Consider using a more generic implementation that can handle different hash functions and lengths and allow the user to choose.

### [L-3] DRAFT OPENZEPPELIN DEPENDENCIES

#### Description:

OpenZeppelin contracts may be considered draft contracts if they have not received adequate security auditing or are liable to change with future development.

#### **Proof Of Concept**

```solidity
File: Well.sol

6: import {ERC20Upgradeable, ERC20PermitUpgradeable} from "ozu/token/ERC20/extensions/draft-ERC20PermitUpgradeable.sol";

```

### [L-4] LOSS OF PRECISION DUE TO ROUNDING

#### **Proof Of Concept**

```solidity
File: functions/ConstantProduct2.sol

99:         reserve = reserves[i] * ratios[j] / ratios[i];

```

```solidity
File: functions/ProportionalLPToken2.sol

23:         underlyingAmounts[1] = lpTokenAmount * reserves[1] / lpTokenSupply;

```

### [L-4] Unify return criteria

#### Description:

In the following functions no value is returned, due to which by default value of return will be 0.We assumed that after the update you return the latest new value. (similar issue here: https://github.com/code-423n4/2021-10-badgerdao-findings/issues/85).

#### **Proof Of Concept**

```solidity
File: Well.sol

84:     function tokens() public pure returns (IERC20[] memory ts) {

88:     function wellFunction() public pure returns (Call memory _wellFunction) {

94:     function pumps() public pure returns (Call[] memory _pumps) {

114:     function wellData() public pure returns (bytes memory) {}

116:     function aquifer() public pure override returns (address) {

123:         returns (

144:     function numberOfTokens() public pure returns (uint256) {

151:     function wellFunctionAddress() public pure returns (address) {

158:     function wellFunctionDataLength() public pure returns (uint256) {

165:     function numberOfPumps() public pure returns (uint256) {

173:     function firstPump() public pure returns (Call memory _pump) {

193:     ) external nonReentrant expire(deadline) returns (uint256 amountOut) {

210:     ) external nonReentrant expire(deadline) returns (uint256 amountOut) {

221:     ) internal returns (uint256 amountOut) {

245:     function getSwapOut(IERC20 fromToken, IERC20 toToken, uint256 amountIn) external view returns (uint256 amountOut) {

271:     ) external nonReentrant expire(deadline) returns (uint256 amountIn) {

311:     function getSwapIn(IERC20 fromToken, IERC20 toToken, uint256 amountOut) external view returns (uint256 amountIn) {

356:     ) external nonReentrant returns (uint256 amountOut) {

379:     function getShiftOut(IERC20 tokenOut) external view returns (uint256 amountOut) {

397:     ) external nonReentrant expire(deadline) returns (uint256 lpAmountOut) {

406:     ) external nonReentrant expire(deadline) returns (uint256 lpAmountOut) {

418:     ) internal returns (uint256 lpAmountOut) {

449:     function getAddLiquidityOut(uint256[] memory tokenAmountsIn) external view returns (uint256 lpAmountOut) {

465:     ) external nonReentrant expire(deadline) returns (uint256[] memory tokenAmountsOut) {

485:     function getRemoveLiquidityOut(uint256 lpAmountIn) external view returns (uint256[] memory tokenAmountsOut) {

501:     ) external nonReentrant expire(deadline) returns (uint256 tokenAmountOut) {

522:     ) external view returns (uint256 tokenAmountOut) {

540:     ) private view returns (uint256 tokenAmountOut) {

553:     ) external nonReentrant expire(deadline) returns (uint256 lpAmountIn) {

575:         returns (uint256 lpAmountIn)

603:     function skim(address recipient) external nonReentrant returns (uint256[] memory skimAmounts) {

615:     function getReserves() external view returns (uint256[] memory reserves) {

624:     function _getReserves(uint256 _numberOfTokens) internal view returns (uint256[] memory reserves) {

645:     function _updatePumps(uint256 _numberOfTokens) internal returns (uint256[] memory reserves) {

684:     ) internal view returns (uint256 lpTokenSupply) {

700:     ) internal view returns (uint256 reserve) {

718:     ) internal view returns (uint256[] memory tokenAmounts) {

734:     ) internal pure returns (uint256 i, uint256 j) {

759:     function _getJ(IERC20[] memory _tokens, IERC20 jToken) internal pure returns (uint256 j) {

778:     ) internal returns (uint256 amountTransferred) {

```

```solidity
File: functions/ConstantProduct2.sol

52:     ) external pure override returns (uint256 lpTokenSupply) {

63:     ) external pure override returns (uint256 reserve) {

69:     function name() external pure override returns (string memory) {

73:     function symbol() external pure override returns (string memory) {

84:     ) external pure override returns (uint256 reserve) {

97:     ) external pure override returns (uint256 reserve) {

```

```solidity
File: libraries/LibBytes.sol

21:     function getBytes32FromBytes(bytes memory data, uint256 i) internal pure returns (bytes32 _bytes) {

78:     function readUint128(bytes32 slot, uint256 n) internal view returns (uint256[] memory reserves) {

```

```solidity
File: pumps/MultiFlowPump.sol

171:     function readLastReserves(address well) public view returns (uint256[] memory reserves) {

203:     ) internal view returns (bytes16 cappedReserve) {

222:     function readLastInstantaneousReserves(address well) public view returns (uint256[] memory reserves) {

239:     function readInstantaneousReserves(address well, bytes memory) public view returns (uint256[] memory emaReserves) {

267:     function readLastCumulativeReserves(address well) public view returns (bytes16[] memory reserves) {

280:     function readCumulativeReserves(address well, bytes memory) public view returns (bytes memory cumulativeReserves) {

285:     function _readCumulativeReserves(address well) internal view returns (bytes16[] memory cumulativeReserves) {

312:     ) public view returns (uint256[] memory twaReserves, bytes memory cumulativeReserves) {

335:     function _getSlotForAddress(address addressValue) internal pure returns (bytes32) {

342:     function _getSlotsOffset(uint256 numberOfReserves) internal pure returns (uint256) {

349:     function _getDeltaTimestamp(uint40 lastTimestamp) internal view returns (uint256) {

```

#### Recommended Mitigation Steps:

add `{return x}` if you want to return the updated value or else remove `returns(uint)` from the `function(){}` if no value you wanted to return

### [L-5] STACK TOO DEEP WHEN COMPILING

#### Description:

The project cannot be compiled due to the “stack too deep” error.

The “stack too deep” error is a limitation of the current code generator. The EVM stack only has 16 slots and that’s sometimes not enough to fit all the local variables, parameters and/or return variables. The solution is to move some of them to memory, which is more expensive but at least makes your code compile.

ref: https://forum.openzeppelin.com/t/stack-too-deep-when-compiling-inline-assembly/11391/6

#### **Proof Of Concept**

```solidity
File: Well.sol

6: import {ERC20Upgradeable, ERC20PermitUpgradeable} from "ozu/token/ERC20/extensions/draft-ERC20PermitUpgradeable.sol";

```

### [L-6] Timestamp Dependence

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as block.timestamp can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of block.timestamp, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers cant rely on the preciseness of the provided timestamp.

#### **Proof Of Concept**

```solidity
File: Well.sol

790:         if (block.timestamp > deadline) {

```

```solidity
File: pumps/MultiFlowPump.sol

88:             _init(slot, uint40(block.timestamp), reserves);

139:         slot.storeLastReserves(uint40(block.timestamp), pumpState.lastReserves);

350:         return uint256(uint40(block.timestamp) - lastTimestamp);

```

### [L-7] Account existence check for low-level calls

#### Description:

Low-level calls `call`/`delegatecall`/`staticcall` return true even if the account called is non-existent (per EVM design). Account existence must be checked prior to calling if needed.

https://github.com/crytic/slither/wiki/Detector-Documentation#low-level-callsn

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

In addition to the zero-address checks, add a check to verify that <address>.code.length > 0

```solidity
File: Aquifer.sol

55:             (bool success, bytes memory returnData) = well.call(initFunctionCall);

```

### [L-8] USE `_SAFEMINT` INSTEAD OF `_MINT`

#### Description:

According to openzepplin’s ERC721, the use of `_mint` is discouraged, use `safeMint` whenever possible.

https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-mint-address-uint256-

#### **Proof Of Concept**

```solidity
File: Well.sol

441:         _mint(recipient, lpAmountOut);

```

#### Recommended Mitigation Steps:

Use `_safeMint` whenever possible instead of `_mint`
