# Gas

# SUMMRY
|      | Issue   | Instance |
|------|---------|----------|  
|[G-01]|>=/<= costs less gas than >/<|34|
|[G-02]|public functions not called by the contract should be declared external instead|6|
|[G-03]|Amounts should be checked for 0 before calling a transfer|5|
|[G-04]|With assembly, .call (bool success) transfer can be done gas-optimized|1|
|[G-05]|Use constants instead of type(uintx).max|5|
|[G-06]|Duplicated require()/if() checks should be refactored to a modifier or function|8|
|[G-07]|x += y costs more gas than x = x + y for state variables|2|
|[G-08]|Remove the initializer modifier|1|
|[G-09]|Replace state variable reads and writes within loops with local variable reads and writes.|2|
|[G-10]|Access mappings directly rather than using accessor function|1|
|[G-11]| Use assembly to emit an event|9|
|[G-12]| uint256(1) and uint256(2) can be used in place of true and false boolean literals to save gas|2|
|[G-13]| multiplications of powers of 2 can be replaced by a left shift operation to save gas|1|
|[G-14]| Using fixed bytes is cheaper than using string|2|
|[G-15]|Not using the named return variables when a function returns, wastes deployment gas|4|
|[G-16]| Use assembly for math (add, sub, mul, div)|8|

## [G-01] >=/<= costs less gas than >/<
The compiler uses opcodes GT and ISZERO for code that uses >, but only requires LT for >=. A similar behaviour applies for >, which uses opcodes LT and ISZERO, but only requires GT for <=.
[Reffrence](https://gist.github.com/CloudEllie/05fda0fdecba2ed7d85f68c709e46c8d#gas-18--costs-less-gas-than-)
```solidity
File: /src/Aquifer.sol
58   if (returnData.length < 68) revert InitFailed("");
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L58


```solidity
File: /src/Well.sol
36   for (uint256 i; i < _tokens.length - 1; ++i) {

37   for (uint256 j = i + 1; j < _tokens.length; ++j) {

101  for (uint256 i; i < _pumps.length; i++) {

233  if (amountOut < minAmountOut) {

363  for (uint256 i; i < _tokens.length; ++i) {

382  for (uint256 i; i < _tokens.length; ++i) {

423  for (uint256 i; i < _tokens.length; ++i) {

429  for (uint256 i; i < _tokens.length; ++i) {

437  if (lpAmountOut < minLpAmountOut) {

452  for (uint256 i; i < _tokens.length; ++i) {

473  for (uint256 i; i < _tokens.length; ++i) {

474  if (tokenAmountsOut[i] < minTokenAmountsOut[i]) {

507  if (tokenAmountOut < minTokenAmountOut) {

579  for (uint256 i; i < _tokens.length; ++i) {

593  for (uint256 i; i < _tokens.length; ++i) {

607  for (uint256 i; i < _tokens.length; ++i) {

633  for (uint256 i; i < reserves.length; ++i) {

662  for (uint256 i; i < _pumps.length; ++i) {

738  for (uint256 k; k < _tokens.length; ++k) {

760  for (j; j < _tokens.length; ++j) {        
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L36



```solidity
File: /src/pumps/MultiFlowPump.sol
84   for (uint256 i; i < numberOfReserves; ++i) {

116  for (uint256 i; i < numberOfReserves; ++i) {

152  for (uint256 i; i < numberOfReserves; ++i) {

178  for (uint256 i; i < numberOfReserves; ++i) {

234  for (uint256 i; i < numberOfReserves; ++i) {

255  for (uint256 i; i < numberOfReserves; ++i) {

301  for (uint256 i; i < cumulativeReserves.length; ++i) {

322  for (uint256 i; i < byteCumulativeReserves.length; ++i) {
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol

```solidity
File: /src/libraries/LibBytes.sol
48   for (uint256 i; i < maxI; ++i) {
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L48


```solidity
File: /src/libraries/LibBytes16.sol
28  for (uint256 i; i < maxI; ++i) {
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes16.sol#L28

```solidity
File: /src/libraries/LibLastReserveBytes.sol
41  for (uint256 i = 1; i < maxI; ++i) {
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol#L41

```solidity
File: /src/libraries/LibWellConstructor.sol
43   for (uint256 i; i < _pumps.length; ++i) {

73   for (uint256 i = 1; i < _tokens.length; ++i) {    
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L43




## [G-02] public functions not called by the contract should be declared external instead
Contracts are [allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents' functions and change the visibility from external to public and can save gas by doing so.

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

## [G-03] Amounts should be checked for 0 before calling a transfer
It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```
In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.

```solidity
File: /src/Well.sol
370   tokenOut.safeTransfer(recipient, amountOut);

447   _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);

512    tokenOut.safeTransfer(recipient, tokenAmountOut);

558    _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);

780    token.safeTransferFrom(from, address(this), amount);
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L370

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
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:
```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```
In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.


```solidity
File: /src/libraries/LibBytes.sol
40  require(reserves[0] <= type(uint128).max, "ByteStorage: too large");

41  require(reserves[1] <= type(uint128).max, "ByteStorage: too large");

49  require(reserves[2 * i] <= type(uint128).max, "ByteStorage: too large");

50  require(reserves[2 * i + 1] <= type(uint128).max, "ByteStorage: too large");

63  require(reserves[reserves.length - 1] <= type(uint128).max, "ByteStorage: too large");
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L40


## [G-06] Duplicated require()/if() checks should be refactored to a modifier or function
sing modifiers or functions can make your code more gas-efficient by reducing the overall number of operations that need to be executed. For example, if you have a complex validation check that involves multiple operations, and you refactor it into a function, then the function can be executed with a single opcode, rather than having to execute each operation separately in multiple locations.
[Reffrence](https://code4rena.com/reports/2023-01-biconomy#g-05-duplicated-requireif-checks-should-be-refactored-to-a-modifier-or-function)
Recommendation
You can consider adding a modifier like below
```
 modifer check (address checkToAddress) {    
     require(checkToAddress != address(0) && checkToAddress != SENTINEL_MODULES, "BSA101");  
      _; 
 }
```

```solidity
File: /src/pumps/MultiFlowPump.sol
225  if (numberOfReserves == 0) {

243  if (numberOfReserves == 0) {

270  if (numberOfReserves == 0) {

289  if (numberOfReserves == 0) {            
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L225


```solidity
File: /src/Aquifer.sol
41  if (salt != bytes32(0)) {

47  if (salt != bytes32(0)) {
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L41


```solidity
File: /src/libraries/LibContractInfo.sol
19  if (success) {

37  if (success) {
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L19

## [G-07] x += y costs more gas than x = x + y for state variables
Mitigation
Replace x += y and x -= y with x = x + y and x = x - y.


```solidity
File: /src/Well.sol
103   dataLoc += PACKED_ADDRESS;

105   dataLoc += ONE_WORD;
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L103    


## [G-08] Remove the initializer modifier

If we can just ensure that the initialize() function could only be called from within the constructor, we shouldn’t need to worry about it getting called again.

```solidity
File: /src/Well.sol
31    function init(string memory name, string memory symbol) public initializer {
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L31

## [G-09] Replace state variable reads and writes within loops with local variable reads and writes.

Reading and writing local variables is cheap, whereas reading and writing state variables that are stored in contract storage is expensive.

```
function badCode() external {                
  for(uint256 i; i < myArray.length; i++) { // state reads
    myCounter++; // state reads and writes
  }        
}
```

```
function goodCode() external {
    uint256 length = myArray.length; // one state read
    uint256 local_mycounter = myCounter; // one state read
    for(uint256 i; i < length; i++) { // local reads
        local_mycounter++; // local reads and writes  
    }
    myCounter = local_mycounter; // one state write
}
```

```solidity
File: /src/Well.sol
        for (uint256 i; i < _pumps.length; i++) {
            _pumps[i].target = _getArgAddress(dataLoc);
            dataLoc += PACKED_ADDRESS;
            pumpDataLength = _getArgUint256(dataLoc);
            dataLoc += ONE_WORD;
            _pumps[i].data = _getArgBytes(dataLoc, pumpDataLength);
            dataLoc += pumpDataLength;
        }
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L101-L108  

## [G-10] Access mappings directly rather than using accessor functions
Saves having to do two JUMP instructions, along with stack setup

```solidity
File: /src/Aquifer.sol
87    return wellImplementations[well];
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L87

## [G-11] Use assembly to emit an event
To efficiently emit events, it's possible to utilize assembly by making use of scratch space and the free memory pointer. This approach has the advantage of potentially avoiding the costs associated with memory expansion.

However, it's important to note that in order to safely optimize this process, it is preferable to cache and restore the free memory pointer.

A good example of such practice can be seen in [Solady's](https://github.com/Vectorized/solady/blob/main/src/tokens/ERC1155.sol#L167) codebase.

[Reffrence](https://gist.github.com/CloudEllie/05fda0fdecba2ed7d85f68c709e46c8d#gas-17-use-assembly-to-emit-an-event)

```solidity
File: /src/Aquifer.sol
76   emit BoreWell(
            well,
            implementation,
            IWell(well).tokens(),
            IWell(well).wellFunction(),
            IWell(well).pumps(),
            IWell(well).wellData()
        );
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L76


```solidity
File: /src/Well.sol
238    emit Swap(fromToken, toToken, amountIn, amountOut, recipient);

305    emit Swap(fromToken, toToken, amountIn, amountOut, recipient);

373    emit Shift(reserves, tokenOut, amountOut, recipient);

443    emit AddLiquidity(tokenAmountsIn, lpAmountOut, recipient);

482   emit RemoveLiquidity(lpAmountIn, tokenAmountsOut, recipient);

516   emit RemoveLiquidityOneToken(lpAmountIn, tokenOut, tokenAmountOut, recipient);

569   emit RemoveLiquidity(lpAmountIn, tokenAmountsOut, recipient);

597   emit Sync(reserves);
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L238

## [G-12] uint256(1) and uint256(2) can be used in place of true and false boolean literals to save gas

```solidity
File: /src/Well.sol
741     foundI = true;

744     foundJ = true;
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L741



## [G-13] multiplications of powers of 2 can be replaced by a left shift operation to save gas
Replace such found multiplications with left shift operations when ensured it is safe to do so. NOTE: This only applies to uint variables!
```solidity
File: /src/functions/ConstantProduct2.sol
65   reserve = lpTokenSupply ** 2;
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L65

## [G-14] Using fixed bytes is cheaper than using string
As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

```solidity
File: /src/functions/ConstantProduct2.sol
69    function name() external pure override returns (string memory) {
        return "Constant Product 2";
      }


73    function symbol() external pure override returns (string memory) {
        return "CP2";
    }

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L69

## [G-15] Not using the named return variables when a function returns, wastes deployment gas
you have the option to define named return variables in function signatures, allowing you to assign values to those variables within the function body and have them automatically returned. However, not using these named return variables or not assigning any values to them does not directly impact the deployment gas cost.

```solidity
File: /src/Aquifer.sol
86    function wellImplementation(address well) external view returns (address implementation) {
        return wellImplementations[well];
    }
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L86



```solidity
File: /src/Well.sol
649            return reserves;

762            return j;
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L762

```solidity
File: /src/libraries/LibBytes.sol
88            return reserves;
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L88

## [G-16] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

```solidity
File: /src/pumps/MultiFlowPump.sol
343        return ((numberOfReserves - 1) / 2 + 1) << 5;
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L343

```solidity
File: /src/functions/ConstantProduct2.sol
65        reserve = lpTokenSupply ** 2;

99        reserve = reserves[i] * ratios[j] / ratios[i];
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L65

```solidity
File: /src/Well.sol
90        uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD;

98        uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD + wellFunctionDataLength();

176        uint256 pumpDataLength = _getArgUint256(dataLoc + PACKED_ADDRESS);

232        amountOut = reserveJBefore - reserves[j];

282        amountIn = reserves[i] - reserveIBefore;
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L90