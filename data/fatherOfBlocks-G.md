**Aquifer.sol**
- L25/86 - The variable in storage wellImplementations does not have a visibility defined, therefore it is public and has its default getter. Therefore, there is extra expense in creating an external function, which is unnecessary.


**Well.sol**
- L36/37/381/382 - It is not necessary to directly consult the length of an array twice, directly a variable could be created in memory and spend less gas.

- L77/735/736 - It is not necessary to set a variable with its default value, since this generates unnecessary extra gas expense.

- L95/97/101 - It is not necessary to call the numberOfPumps() function twice, you could directly create a variable in memory and save the cost of the double call. In the for loop, instead of querying the length, directly use this variable in memory.
