
# VULN 1 

## [GAS] Use != 0 instead of > 0 for unsigned integer comparison
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 609 at 2023-07-basin/src/Well.sol:

            if (skimAmounts[i] > 0) {


Found in line 119 at 2023-07-basin/src/pumps/MultiFlowPump.sol:

                pumpState.lastReserves[i], (reserves[i] > 0 ? reserves[i] : 1).fromUIntToLog2(), blocksPassed

------------------------------------------------------------------------ 

### Mitigation 

Use != 0 instead of > 0 for unsigned integer comparison.










# VULN 2 

## [GAS] Use shift Right/Left instead of division/multiplication if possible
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 343 at 2023-07-basin/src/pumps/MultiFlowPump.sol:

        return ((numberOfReserves - 1) / 2 + 1) << 5;


Found in line 39 at 2023-07-basin/src/libraries/LibLastReserveBytes.sol:

            uint256 maxI = n / 2; // number of fully-packed slots


Found in line 97 at 2023-07-basin/src/libraries/LibLastReserveBytes.sol:

                iByte = (i - 1) / 2 * 32;


Found in line 46 at 2023-07-basin/src/libraries/LibBytes.sol:

            uint256 maxI = reserves.length / 2; // number of fully-packed slots


Found in line 96 at 2023-07-basin/src/libraries/LibBytes.sol:

            iByte = (i - 1) / 2 * 32;


Found in line 26 at 2023-07-basin/src/libraries/LibBytes16.sol:

            uint256 maxI = reserves.length / 2; // number of fully-packed slots


Found in line 73 at 2023-07-basin/src/libraries/LibBytes16.sol:

            iByte = (i - 1) / 2 * 32;

------------------------------------------------------------------------ 

### Mitigation 

Use shift Right/Left instead of division/multiplication if possible.










# VULN 3 

## [GAS] ++i/i++ should be unchecked{++i}/unchecked{i++} and ++i costs less gas than i++
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 101 at 2023-07-basin/src/Well.sol:

        for (uint256 i; i < _pumps.length; i++) {

------------------------------------------------------------------------ 

### Mitigation 

++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for and while-loops. Moreover ++i costs less gas than i++, especially when its used in for-loops (--i/i-- too).










# VULN 4 

## [GAS] Use Custom Errors
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 617 at 2023-07-basin/src/Well.sol:

        require(!_reentrancyGuardEntered(), "ReentrancyGuard: reentrant call");


Found in line 58 at 2023-07-basin/src/Aquifer.sol:

                if (returnData.length < 68) revert InitFailed("");


Found in line 40 at 2023-07-basin/src/libraries/LibBytes.sol:

            require(reserves[0] <= type(uint128).max, "ByteStorage: too large");


Found in line 41 at 2023-07-basin/src/libraries/LibBytes.sol:

            require(reserves[1] <= type(uint128).max, "ByteStorage: too large");


Found in line 49 at 2023-07-basin/src/libraries/LibBytes.sol:

                require(reserves[2 * i] <= type(uint128).max, "ByteStorage: too large");


Found in line 50 at 2023-07-basin/src/libraries/LibBytes.sol:

                require(reserves[2 * i + 1] <= type(uint128).max, "ByteStorage: too large");


Found in line 63 at 2023-07-basin/src/libraries/LibBytes.sol:

                require(reserves[reserves.length - 1] <= type(uint128).max, "ByteStorage: too large");

------------------------------------------------------------------------ 

### Mitigation 

Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.










# VULN 5 

## [GAS] Use calldata instead of memory for function arguments that do not get mutated
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 31 at 2023-07-basin/src/Well.sol:

    function init(string memory name, string memory symbol) public initializer {


Found in line 449 at 2023-07-basin/src/Well.sol:

    function getAddLiquidityOut(uint256[] memory tokenAmountsIn) external view returns (uint256 lpAmountOut) {


Found in line 632 at 2023-07-basin/src/Well.sol:

    function _setReserves(IERC20[] memory _tokens, uint256[] memory reserves) internal {


Found in line 759 at 2023-07-basin/src/Well.sol:

    function _getJ(IERC20[] memory _tokens, IERC20 jToken) internal pure returns (uint256 j) {


Found in line 146 at 2023-07-basin/src/pumps/MultiFlowPump.sol:

    function _init(bytes32 slot, uint40 lastTimestamp, uint256[] memory reserves) internal {


Found in line 239 at 2023-07-basin/src/pumps/MultiFlowPump.sol:

    function readInstantaneousReserves(address well, bytes memory) public view returns (uint256[] memory emaReserves) {


Found in line 280 at 2023-07-basin/src/pumps/MultiFlowPump.sol:

    function readCumulativeReserves(address well, bytes memory) public view returns (bytes memory cumulativeReserves) {


Found in line 19 at 2023-07-basin/src/libraries/LibLastReserveBytes.sol:

    function storeLastReserves(bytes32 slot, uint40 lastTimestamp, bytes16[] memory reserves) internal {


Found in line 21 at 2023-07-basin/src/libraries/LibBytes.sol:

    function getBytes32FromBytes(bytes memory data, uint256 i) internal pure returns (bytes32 _bytes) {


Found in line 37 at 2023-07-basin/src/libraries/LibBytes.sol:

    function storeUint128(bytes32 slot, uint256[] memory reserves) internal {


Found in line 87 at 2023-07-basin/src/libraries/LibWellConstructor.sol:

    function encodeCall(address target, bytes memory data) public pure returns (Call memory) {


Found in line 19 at 2023-07-basin/src/libraries/LibBytes16.sol:

    function storeBytes16(bytes32 slot, bytes16[] memory reserves) internal {

------------------------------------------------------------------------ 

### Mitigation 

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.










# VULN 6 

## [GAS] Cache array length outside of loop
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 36 at 2023-07-basin/src/Well.sol:

        for (uint256 i; i < _tokens.length - 1; ++i) {


Found in line 37 at 2023-07-basin/src/Well.sol:

            for (uint256 j = i + 1; j < _tokens.length; ++j) {


Found in line 101 at 2023-07-basin/src/Well.sol:

        for (uint256 i; i < _pumps.length; i++) {


Found in line 363 at 2023-07-basin/src/Well.sol:

        for (uint256 i; i < _tokens.length; ++i) {


Found in line 382 at 2023-07-basin/src/Well.sol:

        for (uint256 i; i < _tokens.length; ++i) {


Found in line 423 at 2023-07-basin/src/Well.sol:

            for (uint256 i; i < _tokens.length; ++i) {


Found in line 429 at 2023-07-basin/src/Well.sol:

            for (uint256 i; i < _tokens.length; ++i) {


Found in line 452 at 2023-07-basin/src/Well.sol:

        for (uint256 i; i < _tokens.length; ++i) {


Found in line 473 at 2023-07-basin/src/Well.sol:

        for (uint256 i; i < _tokens.length; ++i) {


Found in line 557 at 2023-07-basin/src/Well.sol:

        for (uint256 i; i < _tokens.length; ++i) {


Found in line 579 at 2023-07-basin/src/Well.sol:

        for (uint256 i; i < _tokens.length; ++i) {


Found in line 593 at 2023-07-basin/src/Well.sol:

        for (uint256 i; i < _tokens.length; ++i) {


Found in line 607 at 2023-07-basin/src/Well.sol:

        for (uint256 i; i < _tokens.length; ++i) {


Found in line 633 at 2023-07-basin/src/Well.sol:

        for (uint256 i; i < reserves.length; ++i) {


Found in line 662 at 2023-07-basin/src/Well.sol:

            for (uint256 i; i < _pumps.length; ++i) {


Found in line 738 at 2023-07-basin/src/Well.sol:

        for (uint256 k; k < _tokens.length; ++k) {


Found in line 760 at 2023-07-basin/src/Well.sol:

        for (j; j < _tokens.length; ++j) {


Found in line 301 at 2023-07-basin/src/pumps/MultiFlowPump.sol:

        for (uint256 i; i < cumulativeReserves.length; ++i) {


Found in line 322 at 2023-07-basin/src/pumps/MultiFlowPump.sol:

        for (uint256 i; i < byteCumulativeReserves.length; ++i) {


Found in line 43 at 2023-07-basin/src/libraries/LibWellConstructor.sol:

        for (uint256 i; i < _pumps.length; ++i) {


Found in line 73 at 2023-07-basin/src/libraries/LibWellConstructor.sol:

        for (uint256 i = 1; i < _tokens.length; ++i) {

------------------------------------------------------------------------ 

### Mitigation 

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).










# VULN 7 

## [GAS] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 173 at 2023-07-basin/src/pumps/MultiFlowPump.sol:

        (uint8 numberOfReserves,, bytes16[] memory bytesReserves) = slot.readLastReserves();


Found in line 224 at 2023-07-basin/src/pumps/MultiFlowPump.sol:

        uint8 numberOfReserves = slot.readNumberOfReserves();


Found in line 242 at 2023-07-basin/src/pumps/MultiFlowPump.sol:

        (uint8 numberOfReserves, uint40 lastTimestamp, bytes16[] memory lastReserves) = slot.readLastReserves();


Found in line 269 at 2023-07-basin/src/pumps/MultiFlowPump.sol:

        uint8 numberOfReserves = slot.readNumberOfReserves();


Found in line 288 at 2023-07-basin/src/pumps/MultiFlowPump.sol:

        (uint8 numberOfReserves, uint40 lastTimestamp, bytes16[] memory lastReserves) = slot.readLastReserves();


Found in line 13 at 2023-07-basin/src/libraries/LibLastReserveBytes.sol:

    function readNumberOfReserves(bytes32 slot) internal view returns (uint8 _numberOfReserves) {


Found in line 21 at 2023-07-basin/src/libraries/LibLastReserveBytes.sol:

        uint8 n = uint8(reserves.length);


Found in line 71 at 2023-07-basin/src/libraries/LibLastReserveBytes.sol:

        returns (uint8 n, uint40 lastTimestamp, bytes16[] memory reserves)


Found in line 52 at 2023-07-basin/src/libraries/LibContractInfo.sol:

    function getDecimals(address _contract) internal view returns (uint8 decimals) {


Found in line 54 at 2023-07-basin/src/libraries/LibContractInfo.sol:

        decimals = success ? abi.decode(data, (uint8)) : 18; // default to 18 decimals

------------------------------------------------------------------------ 

### Mitigation 

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.










# VULN 8 

## [GAS] Public Functions to external
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 116 at 2023-07-basin/src/Well.sol:

    function aquifer() public pure override returns (address) {

------------------------------------------------------------------------ 

### Mitigation 

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.










# VULN 9 

## [GAS] <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 103 at 2023-07-basin/src/Well.sol:

            dataLoc += PACKED_ADDRESS;


Found in line 105 at 2023-07-basin/src/Well.sol:

            dataLoc += ONE_WORD;


Found in line 107 at 2023-07-basin/src/Well.sol:

            dataLoc += pumpDataLength;


Found in line 226 at 2023-07-basin/src/Well.sol:

        reserves[i] += amountIn;


Found in line 250 at 2023-07-basin/src/Well.sol:

        reserves[i] += amountIn;

------------------------------------------------------------------------ 

### Mitigation 

When you use the += operator on a state variable, the EVM has to perform three operations: load the current value of the state variable, add the new value to it, and then store the result back in the state variable. On the other hand, when you use the = operator and then add the values separately, the EVM only needs to perform two operations: load the current value of the state variable and add the new value to it. Better use <x> = <x> + <y> For State Variables.










# VULN 10 

## [GAS] Use hardcode address instead address(this)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 194 at 2023-07-basin/src/Well.sol:

        fromToken.safeTransferFrom(msg.sender, address(this), amountIn);


Found in line 303 at 2023-07-basin/src/Well.sol:

        fromToken.safeTransferFrom(msg.sender, address(this), amountIn);


Found in line 364 at 2023-07-basin/src/Well.sol:

            reserves[i] = _tokens[i].balanceOf(address(this));


Found in line 383 at 2023-07-basin/src/Well.sol:

            reserves[i] = _tokens[i].balanceOf(address(this));


Found in line 431 at 2023-07-basin/src/Well.sol:

                _tokens[i].safeTransferFrom(msg.sender, address(this), tokenAmountsIn[i]);


Found in line 594 at 2023-07-basin/src/Well.sol:

            reserves[i] = _tokens[i].balanceOf(address(this));


Found in line 608 at 2023-07-basin/src/Well.sol:

            skimAmounts[i] = _tokens[i].balanceOf(address(this)) - reserves[i];


Found in line 634 at 2023-07-basin/src/Well.sol:

            if (reserves[i] > _tokens[i].balanceOf(address(this))) revert InvalidReserves();


Found in line 779 at 2023-07-basin/src/Well.sol:

        uint256 balanceBefore = token.balanceOf(address(this));


Found in line 780 at 2023-07-basin/src/Well.sol:

        token.safeTransferFrom(from, address(this), amount);


Found in line 781 at 2023-07-basin/src/Well.sol:

        amountTransferred = token.balanceOf(address(this)) - balanceBefore;


Found in line 69 at 2023-07-basin/src/Aquifer.sol:

        if (IWell(well).aquifer() != address(this)) {

------------------------------------------------------------------------ 

### Mitigation 

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.










# VULN 11 

## [GAS] Do not calculate constants
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 28 at 2023-07-basin/src/Well.sol:

    uint256 constant ONE_WORD_PLUS_PACKED_ADDRESS = 52; // For gas efficiency purposes


Found in line 29 at 2023-07-basin/src/Well.sol:

    bytes32 constant RESERVES_STORAGE_SLOT = bytes32(uint256(keccak256("reserves.storage.slot")) - 1);


Found in line 78 at 2023-07-basin/src/Well.sol:

    uint256 constant LOC_TOKENS_COUNT = LOC_AQUIFER_ADDR + PACKED_ADDRESS;


Found in line 79 at 2023-07-basin/src/Well.sol:

    uint256 constant LOC_WELL_FUNCTION_ADDR = LOC_TOKENS_COUNT + ONE_WORD;


Found in line 80 at 2023-07-basin/src/Well.sol:

    uint256 constant LOC_WELL_FUNCTION_DATA_LENGTH = LOC_WELL_FUNCTION_ADDR + PACKED_ADDRESS;


Found in line 81 at 2023-07-basin/src/Well.sol:

    uint256 constant LOC_PUMPS_COUNT = LOC_WELL_FUNCTION_DATA_LENGTH + ONE_WORD;


Found in line 82 at 2023-07-basin/src/Well.sol:

    uint256 constant LOC_VARIABLE = LOC_PUMPS_COUNT + ONE_WORD;

------------------------------------------------------------------------ 

### Mitigation 

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.










# VULN 12 

## [GAS] >= costs less gas than >
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 58 at 2023-07-basin/src/Aquifer.sol:

                if (returnData.length < 68) revert InitFailed("");

------------------------------------------------------------------------ 

### Mitigation 

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas.










# VULN 13 

## [GAS] With assembly, .call (bool success) transfer can be done gas-optimized
------------------------------------------------------------------------ 

### Proof of concept 

Found in 2023-07-basin/src/Aquifer.sol (function boreWell():

                well = implementation.clone();

            }

        }



        if (initFunctionCall.length > 0) {

            (bool success, bytes memory returnData) = well.call(initFunctionCall);

            if (!success) {

                // Next 5 lines are based on https://ethereum.stackexchange.com/a/83577

                if (returnData.length < 68) revert InitFailed("");

                assembly {

                    returnData := add(returnData, 0x04)



Found in 2023-07-basin/src/libraries/LibContractInfo.sol (function getSymbol(address _contract) internal view returns (string memory symbol) {):

     * @param _contract The contract to get the symbol of

     * @return symbol The symbol of the contract

     * @dev if the contract does not have a symbol function, the first 4 bytes of the address are returned

     */

    function getSymbol(address _contract) internal view returns (string memory symbol) {

        (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("symbol()"));

        symbol = new string(4);

        if (success) {

            symbol = abi.decode(data, (string));

        } else {

            assembly {



Found in 2023-07-basin/src/libraries/LibContractInfo.sol (function getName(address _contract) internal view returns (string memory name) {):

     * @param _contract The contract to get the name of

     * @return name The name of the contract

     * @dev if the contract does not have a name function, the first 8 bytes of the address are returned

     */

    function getName(address _contract) internal view returns (string memory name) {

        (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("name()"));

        name = new string(8);

        if (success) {

            name = abi.decode(data, (string));

        } else {

            assembly {



Found in 2023-07-basin/src/libraries/LibContractInfo.sol (function getDecimals(address _contract) internal view returns (uint8 decimals) {):

     * @param _contract The contract to get the decimals of

     * @return decimals The decimals of the contract

     * @dev if the contract does not have a decimals function, 18 is returned

     */

    function getDecimals(address _contract) internal view returns (uint8 decimals) {

        (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("decimals()"));

        decimals = success ? abi.decode(data, (uint8)) : 18; // default to 18 decimals

    }

}


------------------------------------------------------------------------ 

### Mitigation 

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.










# VULN 14 

## [GAS] Empty blocks should be removed or emit something
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 114 at 2023-07-basin/src/Well.sol:

    function wellData() public pure returns (bytes memory) {}


Found in line 656 at 2023-07-basin/src/Well.sol:

            try IPump(_pump.target).update(reserves, _pump.data) {}


Found in line 664 at 2023-07-basin/src/Well.sol:

                try IPump(_pumps[i].target).update(reserves, _pumps[i].data) {}


Found in line 27 at 2023-07-basin/src/Aquifer.sol:

    constructor() ReentrancyGuard() {}

------------------------------------------------------------------------ 

### Mitigation 

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}).










# VULN 15 

## [GAS] require() or revert() statements that check input arguments should be at the top of the function (Also restructured some if’s)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 49 at 2023-07-basin/src/libraries/LibBytes.sol:

                require(reserves[2 * i] <= type(uint128).max, "ByteStorage: too large");


Found in line 50 at 2023-07-basin/src/libraries/LibBytes.sol:

                require(reserves[2 * i + 1] <= type(uint128).max, "ByteStorage: too large");


Found in line 63 at 2023-07-basin/src/libraries/LibBytes.sol:

                require(reserves[reserves.length - 1] <= type(uint128).max, "ByteStorage: too large");

------------------------------------------------------------------------ 

### Mitigation 

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.
