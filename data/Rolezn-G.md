## GAS Summary<a name="GAS Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#gas1-consider-activating-viair-for-deploying) | Consider activating `via-ir` for deploying | 1 | 250 |
| [GAS&#x2011;2](#gas2-use-assembly-to-emit-events) | Use assembly to emit events | 8 | 304 |
| [GAS&#x2011;3](#gas3-counting-down-in-for-statements-is-more-gas-efficient) | Counting down in `for` statements is more gas efficient | 33 | 8481 |
| [GAS&#x2011;4](#gas4-empty-blocks-should-be-removed-or-emit-something) | Empty Blocks Should Be Removed Or Emit Something | 3 | 33 |
| [GAS&#x2011;5](#gas5-using-delete-statement-can-save-gas) | Using `delete` statement can save gas | 1 | 8 |
| [GAS&#x2011;6](#gas6-use-assembly-to-write-address-storage-values) | Use `assembly` to write address storage values | 1 | 74 |
| [GAS&#x2011;7](#gas7-multiple-accesses-of-a-mappingarray-should-use-a-local-variable-cache) | Multiple accesses of a mapping/array should use a local variable cache | 2 | 160 |
| [GAS&#x2011;8](#gas17-using-xor-^-and-and-&-bitwise-equivalents) | Using XOR (^) and AND (&) bitwise equivalents | 26 | 338 |

Total: 75 contexts over 8 issues

## Gas Optimizations

### <a href="#gas-summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> Consider activating `via-ir` for deploying

The IR-based code generator was introduced with an aim to not only allow code generation to be more transparent and auditable but also to enable more powerful optimization passes that span across functions.

You can enable it on the command line using `--via-ir` or with the option `{"viaIR": true}`.

This will take longer to compile, but you can just simple test it before deploying and if you got a better benchmark then you can add --via-ir to your deploy command

More on: https://docs.soliditylang.org/en/v0.8.17/ir-breaking-changes.html




### <a href="#gas-summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Use assembly to emit events

We can use assembly to emit events efficiently by utilizing `scratch space` and the `free memory pointer`. This will allow us to potentially avoid memory expansion costs.
Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

For example, for a generic `emit` event for `eventSentAmountExample`:
```solidity
// uint256 id, uint256 value, uint256 amount
emit eventSentAmountExample(id, value, amount);
```

We can use the following assembly emit events:

```solidity
        assembly {
            let memptr := mload(0x40)
            mstore(0x00, calldataload(0x44))
            mstore(0x20, calldataload(0xa4))
            mstore(0x40, amount)
            log1(
                0x00,
                0x60,
                // keccak256("eventSentAmountExample(uint256,uint256,uint256)")
                0xa622cf392588fbf2cd020ff96b2f4ebd9c76d7a4bc7f3e6b2f18012312e76bc3
            )
            mstore(0x40, memptr)
        }
```

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: Well.sol

238: emit Swap(fromToken, toToken, amountIn, amountOut, recipient);

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L238

```solidity
File: Well.sol

305: emit Swap(fromToken, toToken, amountIn, amountOut, recipient);

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L305

```solidity
File: Well.sol

373: emit Shift(reserves, tokenOut, amountOut, recipient);

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L373

```solidity
File: Well.sol

443: emit AddLiquidity(tokenAmountsIn, lpAmountOut, recipient);

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L443

```solidity
File: Well.sol

482: emit RemoveLiquidity(lpAmountIn, tokenAmountsOut, recipient);

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L482

```solidity
File: Well.sol

516: emit RemoveLiquidityOneToken(lpAmountIn, tokenOut, tokenAmountOut, recipient);

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L516

```solidity
File: Well.sol

569: emit RemoveLiquidity(lpAmountIn, tokenAmountsOut, recipient);

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L569

```solidity
File: Well.sol

597: emit Sync(reserves);

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L597



</details>





### <a href="#gas-summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Counting down in `for` statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: Well.sol

36: for (uint256 i; i < _tokens.length - 1; ++i) {
37: for (uint256 j = i + 1; j < _tokens.length; ++j) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L36-L37

```solidity
File: Well.sol

101: for (uint256 i; i < _pumps.length; i++) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L101

```solidity
File: Well.sol

363: for (uint256 i; i < _tokens.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L363

```solidity
File: Well.sol

382: for (uint256 i; i < _tokens.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L382

```solidity
File: Well.sol

423: for (uint256 i; i < _tokens.length; ++i) {
423: for (uint256 i; i < _tokens.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L423

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L423



```solidity
File: Well.sol

452: for (uint256 i; i < _tokens.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L452

```solidity
File: Well.sol

473: for (uint256 i; i < _tokens.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L473

```solidity
File: Well.sol

557: for (uint256 i; i < _tokens.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L557

```solidity
File: Well.sol

579: for (uint256 i; i < _tokens.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L579

```solidity
File: Well.sol

593: for (uint256 i; i < _tokens.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L593

```solidity
File: Well.sol

607: for (uint256 i; i < _tokens.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L607

```solidity
File: Well.sol

633: for (uint256 i; i < reserves.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L633

```solidity
File: Well.sol

662: for (uint256 i; i < _pumps.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L662

```solidity
File: Well.sol

738: for (uint256 k; k < _tokens.length; ++k) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L738

```solidity
File: Well.sol

760: for (j; j < _tokens.length; ++j) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L760

```solidity
File: LibBytes.sol

48: for (uint256 i; i < maxI; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibBytes.sol#L48

```solidity
File: LibBytes.sol

92: for (uint256 i = 1; i <= n; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibBytes.sol#L92

```solidity
File: LibBytes16.sol

28: for (uint256 i; i < maxI; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibBytes16.sol#L28

```solidity
File: LibBytes16.sol

69: for (uint256 i = 1; i <= n; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibBytes16.sol#L69

```solidity
File: LibLastReserveBytes.sol

41: for (uint256 i = 1; i < maxI; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibLastReserveBytes.sol#L41

```solidity
File: LibLastReserveBytes.sol

93: for (uint256 i = 3; i <= n; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibLastReserveBytes.sol#L93

```solidity
File: LibWellConstructor.sol

43: for (uint256 i; i < _pumps.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibWellConstructor.sol#L43

```solidity
File: LibWellConstructor.sol

73: for (uint256 i = 1; i < _tokens.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibWellConstructor.sol#L73

```solidity
File: MultiFlowPump.sol

84: for (uint256 i; i < numberOfReserves; ++i) {
84: for (uint256 i; i < numberOfReserves; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L84

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L84



```solidity
File: MultiFlowPump.sol

152: for (uint256 i; i < numberOfReserves; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L152

```solidity
File: MultiFlowPump.sol

178: for (uint256 i; i < numberOfReserves; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L178

```solidity
File: MultiFlowPump.sol

234: for (uint256 i; i < numberOfReserves; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L234

```solidity
File: MultiFlowPump.sol

255: for (uint256 i; i < numberOfReserves; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L255

```solidity
File: MultiFlowPump.sol

301: for (uint256 i; i < cumulativeReserves.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L301

```solidity
File: MultiFlowPump.sol

322: for (uint256 i; i < byteCumulativeReserves.length; ++i) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L322



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }
    
    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }
    
    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |



### <a href="#gas-summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Empty Blocks Should Be Removed Or Emit Something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}})

#### <ins>Proof Of Concept</ins>

```solidity
File: Well.sol

114: function wellData() public pure returns (bytes memory) {}
```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L114

```solidity
File: Well.sol

656: try IPump(_pump.target).update(reserves, _pump.data) {}
```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L656

```solidity
File: Well.sol

664: try IPump(_pumps[i].target).update(reserves, _pumps[i].data) {}
```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L664






### <a href="#gas-summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Using `delete` statement can save gas

#### <ins>Proof Of Concept</ins>

```solidity
File: Well.sol

77: uint256 constant LOC_AQUIFER_ADDR = 0;

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L77





### <a href="#gas-summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Use `assembly` to write address storage values

#### <ins>Proof Of Concept</ins>

```solidity
File: LibBytes.sol

24: _bytes = ZERO_BYTES;
```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibBytes.sol#L24





### <a href="#gas-summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `someMap[someIndex]`, save its reference like this: `SomeStruct storage someStruct = someMap[someIndex]` and use it.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: LibWellConstructor.sol

74: name = string.concat(name, ":", LibContractInfo.getSymbol(address(_tokens[i])));
75: symbol = string.concat(symbol, LibContractInfo.getSymbol(address(_tokens[i])));

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibWellConstructor.sol#L74-L75



</details>







### <a href="#gas-summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: Well.sol

739: if (iToken == _tokens[k]) {
742: } else if (jToken == _tokens[k]) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L739

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L742



```solidity
File: Well.sol

761: if (jToken == _tokens[j]) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/Well.sol#L761

```solidity
File: ConstantProduct2.sol

66: reserve = LibMath.roundUpDiv(reserve, reserves[j == 1 ? 0 : 1] * EXP_PRECISION);

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/functions/ConstantProduct2.sol#L66

```solidity
File: ConstantProduct2.sol

85: uint256 i = j == 1 ? 0 : 1;

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/functions/ConstantProduct2.sol#L85

```solidity
File: ConstantProduct2.sol

98: uint256 i = j == 1 ? 0 : 1;

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/functions/ConstantProduct2.sol#L98

```solidity
File: LibBytes.sol

39: if (reserves.length == 2) {
62: if (reserves.length & 1 == 1) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibBytes.sol#L39

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibBytes.sol#L62



```solidity
File: LibBytes.sol

83: if (n == 2) {
98: if (i & 1 == 1) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibBytes.sol#L83

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibBytes.sol#L98



```solidity
File: LibBytes16.sol

21: if (reserves.length == 2) {
40: if (reserves.length & 1 == 1) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibBytes16.sol#L21

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibBytes16.sol#L40



```solidity
File: LibBytes16.sol

60: if (n == 2) {
75: if (i & 1 == 1) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibBytes16.sol#L60

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibBytes16.sol#L75



```solidity
File: LibLastReserveBytes.sol

22: if (n == 1) {
53: if (reserves.length & 1 == 1) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibLastReserveBytes.sol#L22

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibLastReserveBytes.sol#L53



```solidity
File: LibLastReserveBytes.sol

80: if (n == 0) return (n, lastTimestamp, reserves);
86: if (n == 1) return (n, lastTimestamp, reserves);
99: if (i & 1 == 1) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibLastReserveBytes.sol#L80

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibLastReserveBytes.sol#L86

https://github.com/code-423n4/2023-07-basin/tree/main/src/libraries/LibLastReserveBytes.sol#L99



```solidity
File: MultiFlowPump.sol

83: if (pumpState.lastTimestamp == 0) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L83

```solidity
File: MultiFlowPump.sol

174: if (numberOfReserves == 0) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L174

```solidity
File: MultiFlowPump.sol

225: if (numberOfReserves == 0) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L225

```solidity
File: MultiFlowPump.sol

243: if (numberOfReserves == 0) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L243

```solidity
File: MultiFlowPump.sol

270: if (numberOfReserves == 0) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L270

```solidity
File: MultiFlowPump.sol

289: if (numberOfReserves == 0) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L289

```solidity
File: MultiFlowPump.sol

319: if (deltaTimestamp == bytes16(0)) {

```

https://github.com/code-423n4/2023-07-basin/tree/main/src/pumps/MultiFlowPump.sol#L319



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |



