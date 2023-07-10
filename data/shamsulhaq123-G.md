|No | Issue |Instances|
|-|:-|:-:|:-:|
|G-1|bytes constants are more eficient than string constans|6
|G-2| Change public function visibility to external|12
|G-3|Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate|1
|G-4|Use do while loops instead of for loops|26
|G-5|Duplicated require()/revert() checks should be refactored to a modifier or functio|1
|G-6|++i costs less gas than i++, especially when it’s used in for-loops (--i/i-- too)|1
|G-7|Duplicated require()/revert() checks should be refactored to a modifier or function|1
|G-8|Use function instead of modifiers (4 instances)|1
|G-9|x += y costs more gas than x = x + y for state variables|1
|G-10|emove the initializer modifier|1
|G-11|Replace state variable reads and writes within loops with local variable reads and writes.|1
|G-12| Using bools for storage incurs overhead|3
|G-13|String literals passed to abi.encode()/abi.encodePacked() should not be split by commas|1
|G-14|Remove or replace unused variables|1
|G-15|Using a positive conditional flow to save a NOT opcode|2
|G-16|Call block.timestamp direclty instead of function|2
|G-17|Access mappings directly rather than using accessor functions|1
|G-18|public functions not called by the contract should be declared external instead|2
|G-19|Amounts should be checked for 0 before calling a transfer|1
|G-20|With assembly, .call (bool success) transfer can be done gas-optimized|1
|G-21|Use constants instead of type(uintx).max|1
|G-22|Duplicated require()/if() checks should be refactored to a modifier or function|3

## [G-1] bytes constants are more eficient than string constans
If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.

```solidity

file: src/functions/ConstantProduct2.sol

69     function name() external pure override returns (string memory) {
        return "Constant Product 2";
    }

    function symbol() external pure override returns (string memory) {
        return "CP2";
75    }


```
https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L69-L75

```solidity

file: src/Aquifer.sol

62  revert InitFailed(abi.decode(returnData, (string)));

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L62

```solidity

file: src/Well.sol

31   function init(string memory name, string memory symbol) public initializer 

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L31

```solidity

file: src/libraries/LibContractInfo.sol

16     function getSymbol(address _contract) internal view returns (string memory symbol) {
        (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("symbol()"));
        symbol = new string(4);
        if (success) {
            symbol = abi.decode(data, (string));
        } else {
            assembly {
                mstore(add(symbol, 0x20), shl(224, shr(128, _contract)))
            }
        }
26    }

34     function getName(address _contract) internal view returns (string memory name) {
        (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("name()"));
        name = new string(8);
        if (success) {
            name = abi.decode(data, (string));
        } else {
            assembly {
                mstore(add(name, 0x20), shl(224, shr(128, _contract)))
            }
        }
44    }


```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L16-L26
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L34-L44

```solidity

file: src/libraries/LibWellConstructor.sol

71     string memory name = LibContractInfo.getSymbol(address(_tokens[0]));
        string memory symbol = name;
        for (uint256 i = 1; i < _tokens.length; ++i) {
            name = string.concat(name, ":", LibContractInfo.getSymbol(address(_tokens[i])));
            symbol = string.concat(symbol, LibContractInfo.getSymbol(address(_tokens[i])));
        }
        name = string.concat(name, " ", LibContractInfo.getName(_wellFunction.target), " Well");
        symbol = string.concat(symbol, LibContractInfo.getSymbol(_wellFunction.target), "w");

        // See {Well.init}.
81        initFunctionCall = abi.encodeWithSignature("init(string,string)", name, symbol);
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L71-L81


## [G-2] Change public function visibility to external
Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.

```solidity

file: src/pumps/MultiFlowPump.sol

171    function readLastReserves(address well) public view returns (uint256[] memory reserves)

222    function readLastInstantaneousReserves(address well) public view returns (uint256[] memory reserves) 

239    function readInstantaneousReserves(address well, bytes memory) public view returns (uint256[] memory emaReserves)

267    function readLastCumulativeReserves(address well) public view returns (bytes16[] memory reserves)

280   function readCumulativeReserves(address well, bytes memory) public view returns (bytes memory cumulativeReserves)  


307 
    function readTwaReserves(
        address well,
        bytes calldata startCumulativeReserves,
        uint256 startTimestamp,
        bytes memory
312    ) public view returns (uint256[] memory twaReserves, bytes memory cumulativeReserves)


```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L171
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L222
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L239
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L267
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L280
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L307-L312

```solidity

file: src/Well.sol

31 function init(string memory name, string memory symbol) public initializer 

84 function tokens() public pure returns (IERC20[] memory ts) {
        ts = _getArgIERC20Array(LOC_VARIABLE, numberOfTokens());
    }

    function wellFunction() public pure returns (Call memory _wellFunction) {
        _wellFunction.target = wellFunctionAddress();
        uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD;
        _wellFunction.data = _getArgBytes(dataLoc, wellFunctionDataLength());
    }

94    function pumps() public pure returns (Call[] memory _pumps) 

114  function wellData() public pure returns (bytes memory) {}

116  function aquifer() public pure override returns (address) 

144     function numberOfTokens() public pure returns (uint256) {
        return _getArgUint256(LOC_TOKENS_COUNT);
    }

    /**
     * @notice Returns the address of the Well Function.
     */
    function wellFunctionAddress() public pure returns (address) {
        return _getArgAddress(LOC_WELL_FUNCTION_ADDR);
    }

    /**
     * @notice Returns the length of the configurable `data` parameter passed during calls to the Well Function.
     */
    function wellFunctionDataLength() public pure returns (uint256) {
        return _getArgUint256(LOC_WELL_FUNCTION_DATA_LENGTH);
    }

    /**
     * @notice Returns the number of Pumps which this Well was initialized with.
     */
    function numberOfPumps() public pure returns (uint256) {
        return _getArgUint256(LOC_PUMPS_COUNT);
167    }

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L31
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L84-L94
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L114-L116
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L144-L167

```solidity

file: src/libraries/LibWellConstructor.sol

67 
    function encodeWellInitFunctionCall(
        IERC20[] memory _tokens,
        Call memory _wellFunction
    ) public view returns (bytes memory initFunctionCall)

87  function encodeCall(address target, bytes memory data) public pure returns (Call memory)

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L67
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L87

## [G-3] Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate

```solidity

file: src/Aquifer.sol

25   mapping(address => address) wellImplementations;

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L25

## [G-4] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity

file: src/pumps/MultiFlowPump.sol

84 for (uint256 i; i < numberOfReserves; ++i)

116   for (uint256 i; i < numberOfReserves; ++i) 

152 for (uint256 i; i < numberOfReserves; ++i) 

178   for (uint256 i; i < numberOfReserves; ++i) 

234 for (uint256 i; i < numberOfReserves; ++i)

255    for (uint256 i; i < numberOfReserves; ++i) 

301   for (uint256 i; i < cumulativeReserves.length; ++i) 

322   for (uint256 i; i < byteCumulativeReserves.length; ++i) 
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L84
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L116
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L152
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L178
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L234
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L255
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L301
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L322

```solidity

file: src/Well.sol

101  for (uint256 i; i < _pumps.length; i++) 

323  for (uint256 i; i < _tokens.length; ++i) 

382  for (uint256 i; i < _tokens.length; ++i) 

423  for (uint256 i; i < _tokens.length; ++i) 

429  for (uint256 i; i < _tokens.length; ++i) 

452  for (uint256 i; i < _tokens.length; ++i)

473  for (uint256 i; i < _tokens.length; ++i) 

557  for (uint256 i; i < _tokens.length; ++i) 

579  for (uint256 i; i < _tokens.length; ++i) 

633  for (uint256 i; i < reserves.length; ++i)

738  for (uint256 k; k < _tokens.length; ++k) 

760  for (j; j < _tokens.length; ++j)
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L101
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L363
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L382
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L423
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L429
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L452
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L473
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L557
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L579
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L633
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L738
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L760

```solidity

file: src/libraries/LibBytes.sol

48    for (uint256 i; i < maxI; ++i) 

92  for (uint256 i = 1; i <= n; ++i)
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L48
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L92

```solidity

file: src/libraries/LibLastReserveBytes.sol

41    for (uint256 i = 1; i < maxI; ++i) 

93      for (uint256 i = 3; i <= n; ++i) 
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol#L41
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol#L93

```solidity

file: libraries/LibWellConstructor.sol

43  for (uint256 i; i < _pumps.length; ++i)

73       for (uint256 i = 1; i < _tokens.length; ++i)
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L43
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L73

## [G-5]  Duplicated require()/revert() checks should be refactored to a modifier or functio

```solidity

file: 

40   require(reserves[0] <= type(uint128).max, "ByteStorage: too large");
41   require(reserves[1] <= type(uint128).max, "ByteStorage: too large");


```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L40-L41

## [G-6] ++i costs less gas than i++, especially when it’s used in for-loops (--i/i-- too)
Saves 5 gas per loop

```solidity

file: src/Well.sol

101  for (uint256 i; i < _pumps.length; i++)


```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L101


## [G-7]  Duplicated require()/revert() checks should be refactored to a modifier or function

```solidity

file: 

71 
        string memory name = LibContractInfo.getSymbol(address(_tokens[0]));
        string memory symbol = name;
        for (uint256 i = 1; i < _tokens.length; ++i) {
            name = string.concat(name, ":", LibContractInfo.getSymbol(address(_tokens[i])));
            symbol = string.concat(symbol, LibContractInfo.getSymbol(address(_tokens[i])));
        }
        name = string.concat(name, " ", LibContractInfo.getName(_wellFunction.target), " Well");
78        symbol = string.concat(symbol, LibContractInfo.getSymbol(_wellFunction.target), "w");

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L71-L78

## [G-8]  Use function instead of modifiers (4 instances)
Deployment. Gas Saved: 177 805
Minumal Method Call. Gas Saved: -389
Average Method Call. Gas Saved: -990
Maximum Method Call. Gas Saved: 1 902

```solidity

file: src/Well.sol

789     modifier expire(uint256 deadline) {
        if (block.timestamp > deadline) {
            revert Expired();
        }
        _;
    }

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L789


## [G-9] x += y costs more gas than x = x + y for state variables
Mitigation
Replace x += y and x -= y with x = x + y and x = x - y.


```solidity
File: /src/Well.sol
103   dataLoc += PACKED_ADDRESS;

105   dataLoc += ONE_WORD;
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L103    


## [G-10] Remove the initializer modifier

If we can just ensure that the initialize() function could only be called from within the constructor, we shouldn’t need to worry about it getting called again.

```solidity
File: /src/Well.sol
31    function init(string memory name, string memory symbol) public initializer {
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L31

## [G-11] Replace state variable reads and writes within loops with local variable reads and writes.
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

## [G-12]   Using bools for storage incurs overhead
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past.

```solidity
file:    src/libraries/LibContractInfo.sol

17      (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("symbol()"));

35      (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("name()"));

53      (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("decimals()"));
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L17

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L35

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L53


## [G-13]   String literals passed to abi.encode()/abi.encodePacked() should not be split by commas
String literals can be split into multiple parts and still be considered as a single string literal. Adding commas between each chunk makes it no longer a single string, and instead multiple strings. EACH new comma costs 21 gas due to stack operations and separate MSTOREs.


```solidity
file:   src/libraries/LibWellConstructor.sol

81      initFunctionCall = abi.encodeWithSignature("init(string,string)", name, symbol);

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L81



## [G-14]  Remove or replace unused variables
Remove or replace unused variables to save deployment gas.
Instances include:

```solidity
file:   src/libraries/LibBytes16.sol

14      bytes32 private constant ZERO_BYTES = bytes32(0);

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes16.sol#L14


## [G-15] Using a positive conditional flow to save a NOT opcode
Estimated savings: 3 gas

```solidity
file:     src/Well.sol

748       if (!foundI) revert InvalidTokens();
749       if (!foundJ) revert InvalidTokens();

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L748

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L749


## [G-16] Call block.timestamp direclty instead of function


```solidity
file:   src/pumps/MultiFlowPump.sol

251     uint256 deltaTimestamp = _getDeltaTimestamp(lastTimestamp);

297     uint256 deltaTimestamp = _getDeltaTimestamp(lastTimestamp);

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L251

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L297

## [G-17] Access mappings directly rather than using accessor functions
Saves having to do two JUMP instructions, along with stack setup

```solidity
File: /src/Aquifer.sol
87    return wellImplementations[well];
```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L87


## [G-18] public functions not called by the contract should be declared external instead
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

## [G-19] Amounts should be checked for 0 before calling a transfer
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

## [G-20] With assembly, .call (bool success) transfer can be done gas-optimized
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

## [G-21] Use constants instead of type(uintx).max
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


## [G-22] Duplicated require()/if() checks should be refactored to a modifier or function
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