## Gas Optimizations

|       | Issue                                                              |
| ----- | :----------------------------------------------------------------- |
| GAS-1 | <ARRAY>.LENGTH SHOULD NOT BE LOOKED UP IN EVERY LOOP OF A FOR-LOOP |
| GAS-2 | USE FUNCTION INSTEAD OF MODIFIERS                                  |
| GAS-3 | OPTIMIZE NFT DELEGATE DEPLOYMENTS BY USING PROXY                   |
| GAS-4 | TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT                |
| GAS-5 | USE BYTES32 INSTEAD OF STRING                                      |

### [GAS-1] <ARRAY>.LENGTH SHOULD NOT BE LOOKED UP IN EVERY LOOP OF A FOR-LOOP

#### Description:

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

[Source](https://code4rena.com/reports/2022-12-backed#g14--arraylength-should-not-be-looked-up-in-every-loop-of-a-for-loop)

#### **Proof Of Concept**

```solidity
File: Well.sol

36:         for (uint256 i; i < _tokens.length - 1; ++i) {

37:             for (uint256 j = i + 1; j < _tokens.length; ++j) {

101:         for (uint256 i; i < _pumps.length; i++) {

363:         for (uint256 i; i < _tokens.length; ++i) {

382:         for (uint256 i; i < _tokens.length; ++i) {

423:             for (uint256 i; i < _tokens.length; ++i) {

429:             for (uint256 i; i < _tokens.length; ++i) {

452:         for (uint256 i; i < _tokens.length; ++i) {

473:         for (uint256 i; i < _tokens.length; ++i) {

557:         for (uint256 i; i < _tokens.length; ++i) {

579:         for (uint256 i; i < _tokens.length; ++i) {

593:         for (uint256 i; i < _tokens.length; ++i) {

607:         for (uint256 i; i < _tokens.length; ++i) {

633:         for (uint256 i; i < reserves.length; ++i) {

662:             for (uint256 i; i < _pumps.length; ++i) {

738:         for (uint256 k; k < _tokens.length; ++k) {

760:         for (j; j < _tokens.length; ++j) {

```

```solidity
File: libraries/LibWellConstructor.sol

43:         for (uint256 i; i < _pumps.length; ++i) {

73:         for (uint256 i = 1; i < _tokens.length; ++i) {

```

```solidity
File: pumps/MultiFlowPump.sol

301:         for (uint256 i; i < cumulativeReserves.length; ++i) {

322:         for (uint256 i; i < byteCumulativeReserves.length; ++i) {

```

### [GAS-2] USE FUNCTION INSTEAD OF MODIFIERS

#### **Proof Of Concept**

```solidity
File: Well.sol

789:     modifier expire(uint256 deadline) {

```

### [GAS-3] OPTIMIZE NFT DELEGATE DEPLOYMENTS BY USING PROXY

#### Description:

The cost of NFT delegate deployments can be significantly reduced by deploying proxies instead of clones of the implementation.

It copies the code of an existing contract (JBTiered721Delegate, JB721TieredGovernance, or JB721GlobalGovernance) and deploys a new contract with the same code. This is a costly operation because each of the three contracts is a big contract with a lot of code. It’ll be much cheaper to deploy non-upgradable proxies instead.

#### **Proof Of Concept**

```solidity
File: Aquifer.sol

42:                 well = implementation.cloneDeterministic(immutableData, salt);

44:                 well = implementation.clone(immutableData);

48:                 well = implementation.cloneDeterministic(salt);

50:                 well = implementation.clone();

```

#### Recommended Mitigation Steps:

Consider using the Clones library from OpenZeppelin–it deploys and absolutely minimal non-upgradable proxy contract. Such proxies, however, cannot be verified on Etherscan. Some more info.x

### [GAS-4] TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT

#### Description:

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

#### **Proof Of Concept**

```solidity
File: Aquifer.sol

41:         if (salt != bytes32(0)) {
                well = implementation.cloneDeterministic(immutableData, salt);
            } else {
                well = implementation.clone(immutableData);
            }

```

```solidity
File: libraries/LibContractInfo.sol

19:     if (success) {
            symbol = abi.decode(data, (string));
        } else {
            assembly {
                mstore(add(symbol, 0x20), shl(224, shr(128, _contract)))
            }
        }

37:     if (success) {
            name = abi.decode(data, (string));
        } else {
            assembly {
                mstore(add(name, 0x20), shl(224, shr(128, _contract)))
            }
        }

```

### [GAS-5] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: Aquifer.sol

62:                 revert InitFailed(abi.decode(returnData, (string)));

```

```solidity
File: Well.sol

31:     function init(string memory name, string memory symbol) public initializer {

31:     function init(string memory name, string memory symbol) public initializer {

```

```solidity
File: functions/ConstantProduct2.sol

69:     function name() external pure override returns (string memory) {

73:     function symbol() external pure override returns (string memory) {

```

```solidity
File: libraries/LibContractInfo.sol

16:     function getSymbol(address _contract) internal view returns (string memory symbol) {

18:         symbol = new string(4);

20:             symbol = abi.decode(data, (string));

34:     function getName(address _contract) internal view returns (string memory name) {

36:         name = new string(8);

38:             name = abi.decode(data, (string));

```

```solidity
File: libraries/LibWellConstructor.sol

71:         string memory name = LibContractInfo.getSymbol(address(_tokens[0]));

72:         string memory symbol = name;

74:             name = string.concat(name, ":", LibContractInfo.getSymbol(address(_tokens[i])));

75:             symbol = string.concat(symbol, LibContractInfo.getSymbol(address(_tokens[i])));

77:         name = string.concat(name, " ", LibContractInfo.getName(_wellFunction.target), " Well");

78:         symbol = string.concat(symbol, LibContractInfo.getSymbol(_wellFunction.target), "w");

81:         initFunctionCall = abi.encodeWithSignature("init(string,string)", name, symbol);

81:         initFunctionCall = abi.encodeWithSignature("init(string,string)", name, symbol);

```
