 
# Gas Optimizations Report

This report focuses on Basin Protocol contest, in context of various improvements that can be made in terms of gas cost.

Some of the opportunities identified for improving gas efficiency throughout the codebase of Basin protocol are categorised into 10 main areas; with further multiple instances in each of the category.

# [G-01] Immutable has more gas efficiency than constant (05 Instances)

Using immutable instead of constant, save more gas due to avoiding storage access for state variables.

Variable values are set through constructor when using immutable, which also eliminates the need of assigning initial values to state variable making them more efficient in terms of gas cost.

Link to the Code:
1.	https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L23
2.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L26
3.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L27
4.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L28
5.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L29


# [G-02] Use uint256(1)/uint256(2) instead for true and false boolean states (04 Instances)

Boolean for storage if not used, saves Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

Link to the Code:
1.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L741
2.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L744
3.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L735
4.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L736


# [G-03] Setting the constructor to payable (02 Instances)

Saves ~13 gas per instance

Link to the Code:
1.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L54
2.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L27


# [G-04] Public functions to external instead (07 Instances)

Functions with public visibility, if not called within the contract needed to be changed external.

Link to the Code:
1.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L221
2.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L239
3.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L267
4.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L280
5.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L307
6.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L31
7.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L87

# [G 05] String literals passed to abi. encodeWithSignature()/abi.encodePacked() should not be split by commas (03 Instances)

String literals can be split into multiple parts and still be considered as a single string literal.
EACH new comma costs 21 gas due to stack operations and separate MSTOREs.
	
Link to the Code:
1.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L44
2.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L52
3.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L81


# [G-06] Use calldata instead of memory for function parameters (21 Instances)

Using calldata in external function does not require data to be stored, which reduced the process time as compared to memory. This in return saves gas during calling the data.

Link to the Code:
1.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L146
2.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L239
3.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L31
4.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L393
5.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L402
6.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L414
7.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L449
8.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L539
9.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L632
10.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L681
11.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L695
12.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L713
13.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L730
14.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L759
15.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L21
16.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L37
17.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes16.sol#L19
18.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol#L19
19.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L13
20.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L36
21.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L67

# [G-07] Cache array length outside of loop (13 Instances)

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

Link to the Code:
1.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L301
2.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L322
3.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L36
4.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L101
5.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L363
6.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L382
7.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L423
8.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L429
9.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L452
10.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L473
11.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L557
12.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L43
13.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L73


# [G 08] Using bools for storage incurs overhead (05 Instances)

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back.
This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.
	
Link to the Code:
1.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L55
2.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L417
3.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L735
4.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L736
5.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L17

# [G-09] Wastage of deployed gas when return is not present for returns function (20 Instances)

Wastage of gas during deployment; when return is absent for named variable when function returns.

Link to the Code:
1.	https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L52
2.	https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L63
3.	https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L84
4.	https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L97
5.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L171
6.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L203
7.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L222
8.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L239
9.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L267
10.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L280
11.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L285
12.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L312
13.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L39
14.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L84
15.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L88
16.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L94
17.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L173
18.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L21
19.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L16
20.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol#L13

# [G-10] Bytes constant consumes less gas as compared to string constant (02 Instances)

Bytes constant are more gas efficient in storing text data than string constant.

Link to the Code:
1.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L71
2.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibWellConstructor.sol#L72
