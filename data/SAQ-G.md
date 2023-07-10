## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Using fixed bytes is cheaper than using string | 4 | - |
| [G-02] | Using delete statement can save gas | 1 | - |
| [G-03] | Not using the named return variables when a function returns, wastes deployment gas | 4 | - |
| [G-04] | Public function not called by the contract should be declared external instead | 5 | - |
| [G-05] | Before transfer of  some functions, we should check some variables for possible gas save | 5 | - |
| [G-06] | Do not calculate state constant variable with keccak256 to save gas | 1 | - |
| [G-07] | Empty blocks should be removed or emit something | 3 | - |
| [G-08] | Duplicated require()/if() checks should be refactored to a modifier or function | 2 | - |
| [G-09] | Use constants instead of type(uintx).max | 5 | - |
| [G-10] | Use Mappings Instead of Arrays | 37 | - |
| [G-11] | Use assembly for math (add, sub, mul, div) | 10 | - |
| [G-12] | Refactor event to avoid emitting empty data | 1 | - |

## Gas Optimizations  

## [G-1] Using fixed bytes is cheaper than using string

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.



```solidity
file: /src/functions/ConstantProduct2.sol

69    function name() external pure override returns (string memory) {
70        return "Constant Product 2";
71    }


73    function symbol() external pure override returns (string memory) {
74        return "CP2";
75    }

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L69


```solidity
File: /src/Aquifer.sol

62                revert InitFailed(abi.decode(returnData, (string)));

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L62


```solidity
file: /src/libraries/LibContractInfo.sol

16    function getSymbol(address _contract) internal view returns (string memory symbol) {

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L16


## [G-2]  Using delete statement can save gas

```solidity
file: /src/Well.sol

77    uint256 constant LOC_AQUIFER_ADDR = 0;

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L77


## [G-3]  Not using the named return variables when a function returns, wastes deployment gas

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

```solidity
file: /src/libraries/LibBytes.sol

88            return reserves;

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L88


### Recommended Code
```solidity

       return (uint40(block.timestamp) - lastTimestamp);

```

## [G-4] Public function not called by the contract should be declared external instead 

Contracts are allowed to override their parents' functions and change the visibility from external to public and can save gas by doing so.

```solidity
file: /src/pumps/MultiFlowPump.sol

222    function readLastInstantaneousReserves(address well) public view returns (uint256[] memory reserves) {

239    function readInstantaneousReserves(address well, bytes memory) public view returns (uint256[] memory emaReserves) {

267    function readLastCumulativeReserves(address well) public view returns (bytes16[] memory reserves) {    

280    function readCumulativeReserves(address well, bytes memory) public view returns (bytes memory cumulativeReserves) {

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L222


```solidity
file: /src/Well.sol

31    function init(string memory name, string memory symbol) public initializer {

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L31


## [G-5] Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:


```solidity
file: /src/Well.sol

194        fromToken.safeTransferFrom(msg.sender, address(this), amountIn);

303        fromToken.safeTransferFrom(msg.sender, address(this), amountIn);

304        toToken.safeTransfer(recipient, amountOut);

558        _tokens[i].safeTransfer(recipient, tokenAmountsOut[i]);

780        token.safeTransferFrom(from, address(this), amount);


```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L194


## [G-6] Do not calculate state constant variable with keccak256 to save gas 

Instead of calculating a statevar with keccake256() every time the contract is made pre calculate them before and only give the result to a constant.

Example:
```solidity
contract MyContract {
    uint256 private constant MY_CONSTANT = 123;

    // ...

    function myFunction() public view returns (uint256) {
        // Use the constant directly instead of calculating it with keccak256
        return MY_CONSTANT * 2;
    }

    // ...
}
```
By defining the constant as a state constant variable and using it directly in the code, we can avoid the additional gas cost of calculating the value using keccak256. This can help reduce the overall gas cost of the smart contract and improve its efficiency

```solidity
file: /src/Well.sol

29    bytes32 constant RESERVES_STORAGE_SLOT = bytes32(uint256(keccak256("reserves.storage.slot")) - 1);

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L29

## [G-7] Empty blocks should be removed or emit something

If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be 
nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if( x ) {}else if(y){ . . . }else{ . . . } => if ( !x) {if(y) { . . . }else{ . . .}}) . 
Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.

```solidity
file: /src/Aquifer.sol

27    constructor() ReentrancyGuard() {}

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L27

```solidity
file: /src/Well.sol

656            try IPump(_pump.target).update(reserves, _pump.data) {}

664            try IPump(_pumps[i].target).update(reserves, _pumps[i].data) {}

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L656


## [G-8] Duplicated require()/if() checks should be refactored to a modifier or function   

   to reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

```solidity
file: /src/pumps/MultiFlowPump.sol

86         if (reserves[i] == 0) return;
153        if (reserves[i] == 0) return;



174        if (numberOfReserves == 0) {
225        if (numberOfReserves == 0) {
243        if (numberOfReserves == 0) {
270        if (numberOfReserves == 0) {         
289        if (numberOfReserves == 0) {       

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L86


## [G-9] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file: /src/libraries/LibBytes.sol

40            require(reserves[0] <= type(uint128).max, "ByteStorage: too large");

41            require(reserves[1] <= type(uint128).max, "ByteStorage: too large");

49            require(reserves[2 * i] <= type(uint128).max, "ByteStorage: too large");

50            require(reserves[2 * i + 1] <= type(uint128).max, "ByteStorage: too large");

63            require(reserves[reserves.length - 1] <= type(uint128).max, "ByteStorage: too large");

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L40

## [10] Use Mappings Instead of Arrays.

When we say "use mappings instead of arrays," we mean that when storing and accessing data in a smart contract, it's more gas-efficient to use mappings instead of arrays. This is because mappings do not require iteration to find a specific element, whereas arrays do.

!!  bellow many array variable is same but used and different function.

```solidity
file: /src/pumps/MultiFlowPump.sol

43        bytes16[] lastReserves;
44        bytes16[] emaReserves;
45        bytes16[] cumulativeReserves;

148        bytes16[] memory byteReserves = new bytes16[](numberOfReserves);

173       (uint8 numberOfReserves,, bytes16[] memory bytesReserves) = slot.readLastReserves();

232        bytes16[] memory byteReserves = slot.readBytes16(numberOfReserves);

241        uint256[] memory reserves = IWell(well).getReserves();

250        bytes16[] memory lastEmaReserves = slot.readBytes16(numberOfReserves);

254        emaReserves = new uint256[](numberOfReserves);

281        bytes16[] memory byteCumulativeReserves = _readCumulativeReserves(well);\

287        uint256[] memory reserves = IWell(well).getReserves();

313        bytes16[] memory byteCumulativeReserves = _readCumulativeReserves(well);
314        bytes16[] memory byteStartCumulativeReserves = abi.decode(startCumulativeReserves, (bytes16[]));

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L43

```solidity
file: /src/Well.sol

222        IERC20[] memory _tokens = tokens();
223        uint256[] memory reserves = _updatePumps(_tokens.length);

246        IERC20[] memory _tokens = tokens();
247        uint256[] memory reserves = _getReserves(_tokens.length);

312        IERC20[] memory _tokens = tokens();
213        uint256[] memory reserves = _getReserves(_tokens.length);

357        IERC20[] memory _tokens = tokens();
358        uint256[] memory reserves = new uint256[](_tokens.length);

380        IERC20[] memory _tokens = tokens();
381        uint256[] memory reserves = new uint256[](_tokens.length);

419        IERC20[] memory _tokens = tokens();
420        uint256[] memory reserves = _updatePumps(_tokens.length);

450        IERC20[] memory _tokens = tokens();
451        uint256[] memory reserves = _getReserves(_tokens.length);

466        IERC20[] memory _tokens = tokens();
467        uint256[] memory reserves = _updatePumps(_tokens.length);

486        IERC20[] memory _tokens = tokens();
487        uint256[] memory reserves = _getReserves(_tokens.length);

502        IERC20[] memory _tokens = tokens();
503        uint256[] memory reserves = _updatePumps(_tokens.length);

523        IERC20[] memory _tokens = tokens();
524        uint256[] memory reserves = _getReserves(_tokens.length);

554        IERC20[] memory _tokens = tokens();
555        uint256[] memory reserves = _updatePumps(_tokens.length);


```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L222


## [G-11] Use assembly for math (add, sub, mul, div)

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

```solidity
file: /src/functions/ProportionalLPToken2.sol

22        underlyingAmounts[0] = lpTokenAmount * reserves[0] / lpTokenSupply;
23        underlyingAmounts[1] = lpTokenAmount * reserves[1] / lpTokenSupply;

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ProportionalLPToken2.sol#L22


```solidity
file: /src/Well.sol

90        uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD;

98        uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD + wellFunctionDataLength();

176        uint256 pumpDataLength = _getArgUint256(dataLoc + PACKED_ADDRESS);

232        amountOut = reserveJBefore - reserves[j];

282        amountIn = reserves[i] - reserveIBefore;



```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L90


## [G-12] Refactor event to avoid emitting empty data


```solidity
file: /src/Aquifer.sol
             
58                if (returnData.length < 68) revert InitFailed(""); ///@audit: "" ,  no data reverted

```
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L58