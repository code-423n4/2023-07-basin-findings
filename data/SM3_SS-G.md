
# summary 

## Gas Optimizations

|  NO  | Issue   | Instance |
|------|---------|----------|  
|[G-01]|  Access mappings directly rather than using accessor function|1|
|[G-02]|  public functions not called by the contract should be declared external instead|6|
|[G-03]|  Amounts should be checked for 0 before calling a transfer|5|
|[G-04]|  With assembly, .call (bool success) transfer can be done gas-optimized|1|
|[G-05]|  Use constants instead of type(uintx).max|5|
|[G-06]|  Duplicated require()/if() checks should be refactored to a modifier or function|8|
|[G-07]|  x += y costs more gas than x = x + y for state variables|2|
|[G-08]|  Remove the initializer modifier|1|
|[G-09]|  Replace state variable reads and writes within loops with local variable reads and writes.|1|
|[G-10]|  Multiple accesses of a mapping/array should use a storage pointer. | 2 |
|[G-11]|  Using bools for storage incurs overhead. | 7 |
|[G-12]|  String literals passed to abi.encode()/abi.encodePacked() should not be split by commas. | 1 |
|[G-13]|  Using a positive conditional flow to save a NOT opcode. | 3 |
|[G-14]|  Call block.timestamp direclty instead of function. | 2 |
|[G-15]|  Using fixed bytes is cheaper than using string | 4 |
|[G-16]|  Not using the named return variables when a function returns, wastes deployment gas | 4 |
|[G-17]|  Use assembly for math (add, sub, mul, div) | 10 |
|[G-18]|  Refactor event to avoid emitting empty data | 1 |


## [G-01] Access mappings directly rather than using accessor functions
when you have a mapping, accessing its values through accessor functions involves an additional layer of indirection, which can incur some gas cost. This is because accessing a value from a mapping typically involves two steps: first, locating the key in the mapping, and second, retrieving the corresponding value.

```solidity
File: /src/Aquifer.sol
87    return wellImplementations[well];
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L87


## [G-02] public functions not called by the contract should be declared external instead
 when a function is declared as public, it is generated with an internal and an external interface. This means the function can be called both internally (within the contract) and externally (by other contracts or accounts). However, if a public function is never called internally and is only expected to be invoked externally, it is more gas-efficient to explicitly declare it as external.


```solidity
File: /src/Well.sol
31    function init(string memory name, string memory symbol) public initializer {
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L31

```solidity
File: /src/pumps/MultiFlowPump.sol
222    function readLastInstantaneousReserves(address well) public view returns (uint256[] memory reserves) {

239     function readInstantaneousReserves(address well, bytes memory) public view returns (uint256[] memory emaReserves) {

267     function readLastCumulativeReserves(address well) public view returns (bytes16[] memory reserves) {

280     function readCumulativeReserves(address well, bytes memory) public view returns (bytes memory cumulativeReserves) {

307   function readTwaReserves(                
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L222

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L239

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L267

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L280

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L307

## [G-03] Amounts should be checked for 0 before calling a transfer
it can be beneficial to check if an amount is zero before invoking a transfer function, such as transfer or safeTransfer, to avoid unnecessary gas costs associated with executing the transfer operation. If the amount is zero, the transfer operation would have no effect, and performing the check can prevent unnecessary gas consumption.


```solidity
File: /src/Well.sol
370   tokenOut.safeTransfer(recipient, amountOut);

447   _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);

512    tokenOut.safeTransfer(recipient, tokenAmountOut);

558    _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);

780    token.safeTransferFrom(from, address(this), amount);
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L370

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L447

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L512

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L558

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L780



## [G-04] With assembly, .call (bool success) transfer can be done gas-optimized
return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.
   (bool success,) = dest.call{value:amount}("");
 bool success; 
 assembly {  
            success := call(gas(), dest, amount, 0, 0)
     }  

```solidity
File: /src/Aquifer.sol
55   (bool success, bytes memory returnData) = well.call(initFunctionCall);
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L55

## [G-05] Use constants instead of type(uintx).max
using type(uintx).max can result in higher gas costs because it involves a runtime operation to calculate the maximum value at runtime. This calculation is performed every time the expression is evaluated.

To save gas, it is recommended to use constants instead of type(uintx).max to represent the maximum value. By declaring a constant with the maximum value, the value is known at compile-time and does not require any runtime calculations.

```solidity
File: /src/libraries/LibBytes.sol
40  require(reserves[0] <= type(uint128).max, "ByteStorage: too large");

41  require(reserves[1] <= type(uint128).max, "ByteStorage: too large");

49  require(reserves[2 * i] <= type(uint128).max, "ByteStorage: too large");

50  require(reserves[2 * i + 1] <= type(uint128).max, "ByteStorage: too large");

63  require(reserves[reserves.length - 1] <= type(uint128).max, "ByteStorage: too large");
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L40

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L41

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L49

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L50

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L63


## [G-06] Duplicated require()/if() checks should be refactored to a modifier or function
it is common to use require() or if() statements to validate certain conditions before executing specific code. However, when the same checks are repeated multiple times within a contract, it can result in redundant code and unnecessary gas consumption.
To save gas and improve code readability and maintainability, it is recommended to refactor duplicated checks into modifiers or functions. By doing so, the checks can be abstracted into reusable code blocks that can be applied to multiple functions within the contract.


```solidity
File: /src/pumps/MultiFlowPump.sol
225  if (numberOfReserves == 0) {

243  if (numberOfReserves == 0) {

270  if (numberOfReserves == 0) {

289  if (numberOfReserves == 0) {            
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L225

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L243

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L270

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L289

```solidity
File: /src/Aquifer.sol
41  if (salt != bytes32(0)) {

47  if (salt != bytes32(0)) {
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L41

https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L47


```solidity
File: /src/libraries/LibContractInfo.sol
19  if (success) {

37  if (success) {
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L19

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L37

## [G-07] x += y costs more gas than x = x + y for state variables
```solidity
File: /src/Well.sol
103   dataLoc += PACKED_ADDRESS;

105   dataLoc += ONE_WORD;
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L103    

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L105


## [G-08] Remove the initializer modifier

If we can just ensure that the initialize() function could only be called from within the constructor, we shouldn’t need to worry about it getting called again.

```solidity
File: /src/Well.sol
31    function init(string memory name, string memory symbol) public initializer {
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L31

## [G-09] Replace state variable reads and writes within loops with local variable reads and writes.
When accessing state variables within loops, each read or write operation incurs additional gas costs due to the storage and memory access operations involved. These costs can accumulate quickly, particularly in loops with a large number of iterations.


```solidity
File: /src/Well.sol
101        for (uint256 i; i < _pumps.length; i++) {
            _pumps[i].target = _getArgAddress(dataLoc);
            dataLoc += PACKED_ADDRESS;
            pumpDataLength = _getArgUint256(dataLoc);
            dataLoc += ONE_WORD;
            _pumps[i].data = _getArgBytes(dataLoc, pumpDataLength);
            dataLoc += pumpDataLength;
        }
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L101-L108  


## [G-10] Multiple accesses of a mapping/array should use a storage pointer
Caching a mapping’s value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable’s reference instead of repeatedly fetching it.

To achieve this, declare a storage pointer for the variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling _pumps[i], save its reference via a storage pointer: _pumpsInfo storage pumpsInfo = _pumps[i] and use the pointer instead.



Cache storage pointers for _pumps[i]

```solidity
file:    libraries/LibWellConstructor.sol

36          function encodeWellImmutableData(
        address _aquifer,
        IERC20[] memory _tokens,
        Call memory _wellFunction,
        Call[] memory _pumps
    ) internal pure returns (bytes memory immutableData) {
        bytes memory packedPumps;
        for (uint256 i; i < _pumps.length; ++i) {
            packedPumps = abi.encodePacked(
                packedPumps,            // previously packed pumps
                _pumps[i].target,       // pump address
                _pumps[i].data.length,  // pump data length
                _pumps[i].data          // pump data (bytes)
            );


74         name = string.concat(name, ":", LibContractInfo.getSymbol(address(_tokens[i])));
            symbol = string.concat(symbol, LibContractInfo.getSymbol(address(_tokens[i])));

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L36-L50

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L74-L75

## [G-11]   Using bools for storage incurs overhead
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past.


```solidity
file:

417     bool feeOnTransfer

735     bool foundI = false;

736     bool foundJ = false;

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L417

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L735

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L736 

```solidity
file:  src/Aquifer.sol

55     (bool success, bytes memory returnData) = well.call(initFunctionCall);

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L55


```solidity
file:    src/libraries/LibContractInfo.sol

17      (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("symbol()"));

35      (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("name()"));

53      (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("decimals()"));
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L17

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L35

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L53


## [G-12]   String literals passed to abi.encode()/abi.encodePacked() should not be split by commas
String literals can be split into multiple parts and still be considered as a single string literal. Adding commas between each chunk makes it no longer a single string, and instead multiple strings. EACH new comma costs 21 gas due to stack operations and separate MSTOREs.


```solidity
file:   src/libraries/LibWellConstructor.sol

81      initFunctionCall = abi.encodeWithSignature("init(string,string)", name, symbol);

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L81


## [G-13] Using a positive conditional flow to save a NOT opcode
Estimated savings: 3 gas

```solidity
file:   src/Aquifer.sol

56      if (!success)

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L56


```solidity
file:     src/Well.sol

748       if (!foundI) revert InvalidTokens();
749       if (!foundJ) revert InvalidTokens();

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L748

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L749


## [G-14] Call block.timestamp direclty instead of function


```solidity
file:   src/pumps/MultiFlowPump.sol

251     uint256 deltaTimestamp = _getDeltaTimestamp(lastTimestamp);

297     uint256 deltaTimestamp = _getDeltaTimestamp(lastTimestamp);

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L251

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L297

## [G-15] Using fixed bytes is cheaper than using string

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.



```solidity
file: src/functions/ConstantProduct2.sol

69    function name() external pure override returns (string memory) {
        return "Constant Product 2";
    }


73    function symbol() external pure override returns (string memory) {
        return "CP2";
    }

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L69-L71

https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L73-L75


```solidity
File: src/Aquifer.sol

62                revert InitFailed(abi.decode(returnData, (string)));

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L62


```solidity
file: src/libraries/LibContractInfo.sol

16    function getSymbol(address _contract) internal view returns (string memory symbol) {

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L16

## [G-16]  Not using the named return variables when a function returns, wastes deployment gas

```solidity
file: /src/pumps/MultiFlowPump.sol

350        return uint256(uint40(block.timestamp) - lastTimestamp);

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L350


```solidity
file: /src/Well.sol

649            return reserves;

762            return j;

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L649

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L762

```solidity
file: /src/libraries/LibBytes.sol

88            return reserves;

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L88


### Recommended Code
```solidity

    return (uint40(block.timestamp) - lastTimestamp);

```

## [G-17] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

```solidity
file: /src/pumps/MultiFlowPump.sol

343        return ((numberOfReserves - 1) / 2 + 1) << 5;

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L343

```solidity
file: /src/functions/ConstantProduct2.sol

65        reserve = lpTokenSupply ** 2;

99        reserve = reserves[i] * ratios[j] / ratios[i];

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L65

https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L99


```solidity
file: /src/functions/ProportionalLPToken2.sol

22        underlyingAmounts[0] = lpTokenAmount * reserves[0] / lpTokenSupply;
23        underlyingAmounts[1] = lpTokenAmount * reserves[1] / lpTokenSupply;

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ProportionalLPToken2.sol#L22

https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ProportionalLPToken2.sol#L23


```solidity
file: /src/Well.sol

90        uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD;

98        uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD + wellFunctionDataLength();

176        uint256 pumpDataLength = _getArgUint256(dataLoc + PACKED_ADDRESS);

232        amountOut = reserveJBefore - reserves[j];

282        amountIn = reserves[i] - reserveIBefore;



```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L90

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L98

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L176

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L232

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L282

## [G-12] Refactor event to avoid emitting empty data


```solidity
file: /src/Aquifer.sol

58                if (returnData.length < 68) revert InitFailed(""); ///@audit: "" ,  no data reverted

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L58