### [G-01] Do not calculate constants variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

** ~200 gas saved  and deploy gas saved: ~22k **

```solidity
src\Well.sol:

  29:     bytes32 constant RESERVES_STORAGE_SLOT = bytes32(uint256(keccak256("reserves.storage.slot")) - 1);

```

```diff
src\Well.sol:

- 29:     bytes32 constant RESERVES_STORAGE_SLOT = bytes32(uint256(keccak256("reserves.storage.slot")) - 1);
+ 29:     bytes32 constant RESERVES_STORAGE_SLOT = 0x4bba01c388049b5ebd30398b65e8ad45b632802c5faf4964e58085ea8ab03715;

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L29


** deploy gas saved: ~55k **
```solidity
src\Well.sol:

  26:     uint256 constant ONE_WORD = 32;
  27:     uint256 constant PACKED_ADDRESS = 20;
  28:     uint256 constant ONE_WORD_PLUS_PACKED_ADDRESS = 52; // For gas efficiency purposes

  55:     /// TYPE NAME LOCATION (CONSTANT)
  77:     uint256 constant LOC_AQUIFER_ADDR = 0;
  78:     uint256 constant LOC_TOKENS_COUNT = LOC_AQUIFER_ADDR + PACKED_ADDRESS;
  79:     uint256 constant LOC_WELL_FUNCTION_ADDR = LOC_TOKENS_COUNT + ONE_WORD;
  80:     uint256 constant LOC_WELL_FUNCTION_DATA_LENGTH = LOC_WELL_FUNCTION_ADDR + PACKED_ADDRESS;
  81:     uint256 constant LOC_PUMPS_COUNT = LOC_WELL_FUNCTION_DATA_LENGTH + ONE_WORD;
  82:     uint256 constant LOC_VARIABLE = LOC_PUMPS_COUNT + ONE_WORD;

```

```diff
src\Well.sol:

  26:     uint256 constant ONE_WORD = 32;
  27:     uint256 constant PACKED_ADDRESS = 20;
  28:     uint256 constant ONE_WORD_PLUS_PACKED_ADDRESS = 52; // For gas efficiency purposes

  77:     uint256 constant LOC_AQUIFER_ADDR = 0;

- 78:     uint256 constant LOC_TOKENS_COUNT = LOC_AQUIFER_ADDR + PACKED_ADDRESS;
          // LOC_TOKENS_COUNT = LOC_AQUIFER_ADDR + PACKED_ADDRESS = 0 + 20
+ 78:     uint256 constant LOC_TOKENS_COUNT = 20; (~190 gas saved)

- 79:     uint256 constant LOC_WELL_FUNCTION_ADDR = LOC_TOKENS_COUNT + ONE_WORD;
          // LOC_WELL_FUNCTION_ADDR = LOC_TOKENS_COUNT + ONE_WORD = 20 + 32
+ 79:     uint256 constant LOC_WELL_FUNCTION_ADDR = 52; (~375 gas saved)

- 80:     uint256 constant LOC_WELL_FUNCTION_DATA_LENGTH = LOC_WELL_FUNCTION_ADDR + PACKED_ADDRESS;
          // LOC_WELL_FUNCTION_DATA_LENGTH = LOC_WELL_FUNCTION_ADDR + PACKED_ADDRESS = 52 + 20
+ 80:     uint256 constant LOC_WELL_FUNCTION_DATA_LENGTH = 72; (~560 gas saved)


- 81:     uint256 constant LOC_PUMPS_COUNT = LOC_WELL_FUNCTION_DATA_LENGTH + ONE_WORD;
          // LOC_PUMPS_COUNT = LOC_WELL_FUNCTION_DATA_LENGTH + ONE_WORD = 72 + 32
+ 81:     uint256 constant LOC_PUMPS_COUNT = 104; (~750 gas saved)


- 82:     uint256 constant LOC_VARIABLE = LOC_PUMPS_COUNT + ONE_WORD;
          // LOC_VARIABLE = LOC_PUMPS_COUNT + ONE_WORD = 104 + 32
+ 82:     uint256 constant LOC_VARIABLE = 136; (~940 gas saved)

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L78-L82



### [G-02] Use abi.encodeWithSelector instead of abi.encodeWithSignature

abi.encodeWithSelector is much cheaper than abi.encodeWithSignature because it doesn't require to compute the selector from the string.

Reference:

https://docs.soliditylang.org/en/v0.8.15/units-and-global-variables.html#abi-encoding-and-decoding-functions
https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison


```solidity
src\libraries\LibContractInfo.sol:

  17:         (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("symbol()"));

  35:         (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("name()"));

  53:         (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("decimals()"));

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L17
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L35
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L53



```solidity
src\libraries\LibWellConstructor.sol:

  81:         initFunctionCall = abi.encodeWithSignature("init(string,string)", name, symbol);

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L81


### [G-03] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}). Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.

```solidity
src\Aquifer.sol:

  27:     constructor() ReentrancyGuard() {}

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L27



```solidity
src\Well.sol:

  114:     function wellData() public pure returns (bytes memory) {}

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L114