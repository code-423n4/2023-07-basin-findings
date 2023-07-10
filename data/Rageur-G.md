## GAS-1: Duplicated require()/revert() checks should be refactored to a modifier or function

### Description

This saves deployment gas.

### Affected file

* Aquifer.sol (Line: 41, 47)
* LibBytes.sol (Line: 39, 62, 83, 98)
* LibBytes16.sol (Line: 21, 40, 60, 75)
* LibContractInfo.sol (Line: 19, 37)
* LibLastReserveBytes.sol (Line: 38, 53, 91, 99)
* MultiFlowPump.sol (Line: 86, 153, 174, 225, 243, 270, 289)
* Well.sol (Line: 95, 424, 430, 648)

## GAS-2: Public functions not called by the contract should be declared external instead

### Description

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

### Affected file

* MultiFlowPump.sol (Line: 222, 239, 267, 307)

## GAS-3: Replace modifier with function

### Description

Modifiers make code more elegant, but cost more than normal functions.

### Affected file

* Well.sol (Line: 789)

## GAS-4: Use ```assembly``` to write address storage values

### Affected file

* MultiFlowPump.sol (Line: 55, 60, 61, 67)

## GAS-5: Use constants instead of type(uintx).max

### Description

It uses more gas in the distribution process and also for each transaction than constant usage.

### Affected file

* LibBytes.sol (Line: 40, 41, 49, 50, 63)

## GAS-6: Using Solidity version 0.8.19 will provide an overall gas optimization

### Description

Use a solidity version of at least 0.8.0 to get overflow protection without SafeMath.
Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining.
Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.
Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings.
Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

### Affected file

* Aquifer.sol (Line: 3)
* ConstantProduct2.sol (Line: 3)
* LibBytes.sol (Line: 3)
* LibBytes16.sol (Line: 3)
* LibContractInfo.sol (Line: 3)
* LibLastReserveBytes.sol (Line: 3)
* LibWellConstructor.sol (Line: 4)
* MultiFlowPump.sol (Line: 3)
* ProportionalLPToken2.sol (Line: 3)
* Well.sol (Line: 3)

## GAS-7: Using immutable on variables that are only set in the constructor and never after (2.1k gas per var)

### Description

Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD.
Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas).

### Affected file

* MultiFlowPump.sol (Line: 36, 37, 38, 39)

## GAS-8: Using private rather than public for constants saves gas

### Description

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function THAT returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table.

### Affected file

* ConstantProduct2.sol (Line: 23)
* Well.sol (Line: 26, 27, 28, 29, 77, 78, 79, 80, 81, 82)

## GAS-9: With assembly, ```.call (bool success)``` transfer can be done gas-optimized

### Description

```return``` data ```(bool success,)``` has to be stored due to EVM architecture, but in a usage in assembly, 'out' and 'outsize' values are given (0,0), this storage disappears and gas optimization is provided.

### Affected file

* Aquifer.sol (Line: 55)
* LibContractInfo.sol (Line: 17, 35, 53)