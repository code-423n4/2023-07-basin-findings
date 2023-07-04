## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol

```solidity
// place this init function after all the state variables, not before
31:    function init(string memory name, string memory symbol) public initializer {

// place this modifier right before the init function since there is no constructor.
789:    modifier expire(uint256 deadline) {
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol

```solidity
// place these public functions before internal ones.
67:    function encodeWellInitFunctionCall(
87:    function encodeCall(address target, bytes memory data) public pure returns (Call memory) {
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol

```solidity
// as by new solidity layout patterns, external functions now come before public functions
120:    function well()
186:    function swapFrom(
203:    function swapFromFeeOnTransfer(
245:    function getSwapOut(IERC20 fromToken, IERC20 toToken, uint256 amountIn) external view returns (uint256 amountOut) {
264:    function swapTo(
311:    function getSwapIn(IERC20 fromToken, IERC20 toToken, uint256 amountOut) external view returns (uint256 amountIn) {
352:    function shift(
379:    function getShiftOut(IERC20 tokenOut) external view returns (uint256 amountOut) {
392:    function addLiquidity(
401:    function addLiquidityFeeOnTransfer(
449:    function getAddLiquidityOut(uint256[] memory tokenAmountsIn) external view returns (uint256 lpAmountOut) {
460:    function removeLiquidity(
485:    function getRemoveLiquidityOut(uint256 lpAmountIn) external view returns (uint256[] memory tokenAmountsOut) {
495:    function removeLiquidityOneToken(
519:    function getRemoveLiquidityOneTokenOut(
548:    function removeLiquidityImbalanced(
572:    function getRemoveLiquidityImbalancedIn(uint256[] calldata tokenAmountsOut)
590:    function sync() external nonReentrant {
603:    function skim(address recipient) external nonReentrant returns (uint256[] memory skimAmounts) {
615:    function getReserves() external view returns (uint256[] memory reserves) {

// internal functions coming before some external ones
215:    function _swapFrom(
296:    function _swapTo(
413:    function _addLiquidity(

// private functions should come last
536:    function _getRemoveLiquidityOneTokenOut(
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol

```solidity
// internal functions coming before public ones.
146:    function _init(bytes32 slot, uint40 lastTimestamp, uint256[] memory reserves) internal {
199:    function _capReserve(
285:    function _readCumulativeReserves(address well) internal view returns (bytes16[] memory cumulativeReserves) {
```

---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol

```solidity
9:library LibWellConstructor {
13:    function encodeWellDeploymentData(
67:    function encodeWellInitFunctionCall(
87:    function encodeCall(address target, bytes memory data) public pure returns (Call memory) {
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol

```solidity
13:    function readNumberOfReserves(bytes32 slot) internal view returns (uint8 _numberOfReserves) {
19:    function storeLastReserves(bytes32 slot, uint40 lastTimestamp, bytes16[] memory reserves) internal {
68:    function readLastReserves(bytes32 slot)
116:    function readBytes(bytes32 slot) internal view returns (bytes32 value) {
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes16.sol

```solidity
19:    function storeBytes16(bytes32 slot, bytes16[] memory reserves) internal {
55:    function readBytes16(bytes32 slot, uint256 n) internal view returns (bytes16[] memory reserves) {
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol

```solidity
// @params missing
21:    function getBytes32FromBytes(bytes memory data, uint256 i) internal pure returns (bytes32 _bytes) {
68:    function readUint128(bytes32 slot, uint256 n) internal view returns (uint256[] memory reserves) {

// @param missing
37:    function storeUint128(bytes32 slot, uint256[] memory reserves) internal {
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol

```solidity
// @params missing
31:    function init(string memory name, string memory symbol) public initializer {
460:    function removeLiquidity(
632:    function _setReserves(IERC20[] memory _tokens, uint256[] memory reserves) internal {

84:    function tokens() public pure returns (IERC20[] memory ts) {
88:    function wellFunction() public pure returns (Call memory _wellFunction) {
94:    function pumps() public pure returns (Call[] memory _pumps) {
114:    function wellData() public pure returns (bytes memory) {}
116:    function aquifer() public pure override returns (address) {
120:    function well()

// @return missing
144:    function numberOfTokens() public pure returns (uint256) {
151:    function wellFunctionAddress() public pure returns (address) {
158:    function wellFunctionDataLength() public pure returns (uint256) {
165:    function numberOfPumps() public pure returns (uint256) {
173:    function firstPump() public pure returns (Call memory _pump) {
615:    function getReserves() external view returns (uint256[] memory reserves) {


// @params and @return missing
186:    function swapFrom(
203:    function swapFromFeeOnTransfer(
215:    function _swapFrom(
245:    function getSwapOut(IERC20 fromToken, IERC20 toToken, uint256 amountIn) external view returns (uint256 amountOut) {
264:    function swapTo(
296:    function _swapTo(
311:    function getSwapIn(IERC20 fromToken, IERC20 toToken, uint256 amountOut) external view returns (uint256 amountIn) {
379:    function getShiftOut(IERC20 tokenOut) external view returns (uint256 amountOut) {
392:    function addLiquidity(
401:    function addLiquidityFeeOnTransfer(
413:    function _addLiquidity(
449:    function getAddLiquidityOut(uint256[] memory tokenAmountsIn) external view returns (uint256 lpAmountOut) {
485:    function getRemoveLiquidityOut(uint256 lpAmountIn) external view returns (uint256[] memory tokenAmountsOut) {
495:    function removeLiquidityOneToken(
519:    function getRemoveLiquidityOneTokenOut(
536:    function _getRemoveLiquidityOneTokenOut(
548:    function removeLiquidityImbalanced(
572:    function getRemoveLiquidityImbalancedIn(uint256[] calldata tokenAmountsOut)
603:    function skim(address recipient) external nonReentrant returns (uint256[] memory skimAmounts) {
624:    function _getReserves(uint256 _numberOfTokens) internal view returns (uint256[] memory reserves) {
645:    function _updatePumps(uint256 _numberOfTokens) internal returns (uint256[] memory reserves) {
681:    function _calcLpTokenSupply(
695:    function _calcReserve(
713:    function _calcLPTokenUnderlying(
730:    function _getIJ(
759:    function _getJ(IERC20[] memory _tokens, IERC20 jToken) internal pure returns (uint256 j) {
774:    function _safeTransferFromFeeOnTransfer(
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol

```solidity
// @params and @return missing
34:    function boreWell(
86:    function wellImplementation(address well) external view returns (address implementation) {
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol

```solidity
41:    struct PumpState {

// @params missing
156:    function _init(bytes32 slot, uint40 lastTimestamp, uint256[] memory reserves) internal {

// @params and @return missing
171:    function readLastReserves(address well) public view returns (uint256[] memory reserves) {
199:    function _capReserve(
222:    function readLastInstantaneousReserves(address well) public view returns (uint256[] memory reserves) {
239:    function readInstantaneousReserves(address well, bytes memory) public view returns (uint256[] memory emaReserves) {
267:    function readLastCumulativeReserves(address well) public view returns (bytes16[] memory reserves) {
280:    function readCumulativeReserves(address well, bytes memory) public view returns (bytes memory cumulativeReserves) {
285:    function _readCumulativeReserves(address well) internal view returns (bytes16[] memory cumulativeReserves) {
307:    function readTwaReserves(
335:    function _getSlotForAddress(address addressValue) internal pure returns (bytes32) {
342:    function _getSlotsOffset(uint256 numberOfReserves) internal pure returns (uint256) {
349:    function _getDeltaTimestamp(uint40 lastTimestamp) internal view returns (uint256) {
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ProportionalLPToken2.sol

```solidity
// @param and @return missing
15:    function calcLPTokenUnderlying(
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol

```solidity
// @param and @return missing
58:    function calcReserve(
79:    function calcReserveAtRatioSwap(

// @return missing
69:    function name() external pure override returns (string memory) {
73:    function symbol() external pure override returns (string memory) {
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE
  
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol

```solidity
//  private and internal `functions` should preppend with `underline`
13:    function encodeWellDeploymentData(
36:    function encodeWellImmutableData(
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol

```solidity
//  private and internal `functions` should preppend with `underline`
13:    function readNumberOfReserves(bytes32 slot) internal view returns (uint8 _numberOfReserves) {
19:    function storeLastReserves(bytes32 slot, uint40 lastTimestamp, bytes16[] memory reserves) internal {
68:    function readLastReserves(bytes32 slot)
116:    function readBytes(bytes32 slot) internal view returns (bytes32 value) {
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol

```solidity
//  private and internal `functions` should preppend with `underline`
16:    function getSymbol(address _contract) internal view returns (string memory symbol) {
34:    function getName(address _contract) internal view returns (string memory name) {
52:    function getDecimals(address _contract) internal view returns (uint8 decimals) {
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes16.sol

```solidity
//  private and internal `functions` should preppend with `underline`
19:    function storeBytes16(bytes32 slot, bytes16[] memory reserves) internal {
55:    function readBytes16(bytes32 slot, uint256 n) internal view returns (bytes16[] memory reserves) {
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol

```solidity
//  private and internal `functions` should preppend with `underline`
21:    function getBytes32FromBytes(bytes memory data, uint256 i) internal pure returns (bytes32 _bytes) {
37:    function storeUint128(bytes32 slot, uint256[] memory reserves) internal {
78:    function readUint128(bytes32 slot, uint256 n) internal view returns (uint256[] memory reserves) {
```
