## G01 - Use  ``<x> += <y>`` instead of ``<x> = <x> + <y>``
In ``Well.sol`` line 504, the reserves of the ``TokenAmountIn`` are updated in the following way:
```reserves[i] = reserves[i] + tokenAmountsIn[i]```
This spends unnecessary gas as this is not an update to a state variable. The code should be as follows:
```reserves[i] += tokenAmountsIn```

## G02 - Declaring and populating a variable with an empty array is unnecessary
In ``Well.sol`` under ``removeLiquidity()`` function, the variable ``tokenAmountsOut`` is initialised with an empty array.
The function ``_calcLPTokenUnderlying()`` already returns an uint256 array, so there is no need for the declaration of an empty uint256 array in line 552.

## G03 - Array length calculation while looping should be cached
For every iteration in the loop, the value ``_tokens.length`` will be fetched afresh. This is expensive. Consider caching the ``_tokens.length`` value outside the loop or use assembly.
These are the following instances of this issue in ``Well.sol`` : lines 44, 45, 113, 410, 433, 486, 497, 528, 561, 670, 692, 708, 725, 757, 788, 877, 902

## G04 - Hardcode the contract’s address instead of using address(this) saves Gas
It is possible to calculate a contract’s address before mainnet deployment. It is less expensive than using ``address(this)`` 
Foundry has a specific tool designed for that: https://book.getfoundry.sh/reference/forge-std/compute-create-address
These are the instances of this issue in ``Well.sol``: lines 212, 343, 411, 434, 501, 709, 726, 758, 921, 922, 923

## G05 - ``>=`` is costs less than  ``>``
When the symbol ``>`` is used the compiler uses ``GT`` and ``ISZERO``, however when ``>=`` is used the compiler only uses ``LT``. This saves 3 gas.

Issue Instances in ``Well.sol``: lines 324, 678, 758, 932.

## G06 - Adding ``payable`` to the constructor saves gas
In ``Aquifier.sol``, the constructor does not have ``payable`` keyword spending unnecessary gas.
Declaring the constructor as payable enables the removal of 10 opcodes from the EVM bytecode generated during creation time. By making the constructor payable, there is no longer a requirement to check if msg.value == 0 during deployment, resulting in a savings of 13 gas without introducing any security risks.

## G07 - Use custom errors rather than revert()/require() strings to save gas
Starting from Solidity version 0.8.4, developers can utilise custom errors, which provide a gas-saving advantage of approximately 50 units per occurrence by eliminating the need for allocating and storing the revert string. Furthermore, omitting the definition of these strings also contributes to reducing the gas consumption during deployment.

