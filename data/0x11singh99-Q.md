# Low-Level and Non-Critical issues

## Summary

### Low Level List

| Number | Issue Details                                                                             | Instances |
| :----: | :---------------------------------------------------------------------------------------- | :-------: |
| [L-01] | Check receiver address for address(0) before transferring or minting any tokens OR Ethers |     3     |
| [L-02] | Check amount for 0 before transferring tokens                                             |     6     |
| [L-03] | Check receiver address for address(0) before transferring or minting any tokens OR Ethers |     1     |

Total 3 Low Level Issues

### Non Critical List

| Number  | Issue Details                                                        | Instances |
| :-----: | :------------------------------------------------------------------- | :-------: |
| [NC-01] | Named return is confusing, use explicit returns                      |     4     |
| [NC-02] | Verify constructor params specially for immutables to check 0 values |     1     |
| [NC-03] | Named params of mapping is more readable                             |     1     |
| [NC-04] | Use ternary operator instead of if else                              |     3     |
| [NC-05] | Some Contracts are not following proper solidity style guide layout  |     1     |

Total 5 Non Critical Issues

# Low-Level issues:

## [L-01] Check receiver address for address(0) before transferring or minting any tokens OR Ethers

_3 Instances_

### Instance#1-2: Make sure `recipient` is not address(0) before transfer ,here in \_swapFrom to fail early

```solidity
File: /src/Well.sol
195:  amountOut = _swapFrom(fromToken, toToken, amountIn, minAmountOut, recipient);
       ...
212:   amountOut = _swapFrom(fromToken, toToken, amountIn, minAmountOut, recipient);
      ...
304:        toToken.safeTransfer(recipient, amountOut);
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L195
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L212
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L304

## [L-02] Check amount for 0 before transferring tokens.

Check before trasferring zero money unnecessary.
_6 Instances_

### Instance#1-6: Make sure `amountIn` is not 0 before transfer

```solidity
File: /src/Well.sol
194:  fromToken.safeTransferFrom(msg.sender, address(this), amountIn);
195:  amountOut = _swapFrom(fromToken, toToken, amountIn, minAmountOut, recipient);
     ...
211:   amountIn = _safeTransferFromFeeOnTransfer(fromToken, msg.sender, amountIn);
212:   amountOut = _swapFrom(fromToken, toToken, amountIn, minAmountOut, recipient);
     ...
303:    fromToken.safeTransferFrom(msg.sender, address(this), amountIn);
304:        toToken.safeTransfer(recipient, amountOut);
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L194-L195
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L211-L212
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L303-L304

## [L-03] Use Check effect interaction flow to avoid reentrancy

_1 Instance_

### Instance#1: Make sure to update mapping `reserves[j]` before transferring tokens to outside

```solidity
File: /src/Well.sol
 371:     tokenOut.safeTransfer(recipient, amountOut);
 372:         reserves[j] -= amountOut;
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L371C5-L372C45

# Non Critcal Issues

## [NC-01] : Named return is confusing, use explicit returns

Return from function explicitly make code readable

_4 Instances_

### Instance# 1-3 :

```solidity
File: src/pumps/MultiFlowPump.sol
171:  function readLastReserves(address well) public view returns (uint256[] memory reserves) {
         ...
199:      function _capReserve(
        bytes16 lastReserve,
        bytes16 reserve,
        bytes16 blocksPassed
203:    ) internal view returns (bytes16 cappedReserve) {
         ...
222:  function readLastInstantaneousReserves(address well) public view returns (uint256[] memory reserves) {

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L171
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L199C3-L203C54
https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L222

### Instance#4 : Doing both explicit return and named return is confusing and ambiguous,do one thing only and explicit return is more better and clear.

```solidity
File: /src/Aquifer.sol
86:  function wellImplementation(address well) external view returns (address implementation) {
87:       return wellImplementations[well];
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L86C4-L87C42

## [NC-02] : Verify constructor params specially for immutables to check 0 values

Verify constructor params for 0 values by mistake otherwise they will be in the contract forever Or make contract re-deploying.
_1 Instance_

### Instance#1:

```solidity
File: src/pumps/MultiFlowPump.sol
 54:  constructor(bytes16 _maxPercentIncrease, bytes16 _maxPercentDecrease, uint256 _blockTime, bytes16 _alpha) {
        LOG_MAX_INCREASE = ABDKMathQuad.ONE.add(_maxPercentIncrease).log_2();
        // _maxPercentDecrease <= 100%
        if (_maxPercentDecrease > ABDKMathQuad.ONE) {
            revert InvalidMaxPercentDecreaseArgument(_maxPercentDecrease);
        }
        LOG_MAX_DECREASE = ABDKMathQuad.ONE.sub(_maxPercentDecrease).log_2();
        BLOCK_TIME = _blockTime;

        // ALPHA <= 1
        if (_alpha > ABDKMathQuad.ONE) {
            revert InvalidAArgument(_alpha);
        }
        ALPHA = _alpha;
 68:   }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L54C3-L68C6

## [NC-03] Named params of mapping is more readable

_1 Instance_

### Instance#1:

```solidity
File: /src/Aquifer.sol
25:    mapping(address => address) wellImplementations;
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L25

## [NC-04] Use ternary operator instead of if else

Using ternary makes code concise and more gas efficient

_3 Instance_

### Instance#1-3:

```solidity
File: /src/Aquifer.sol

40:   if (immutableData.length > 0) {
            if (salt != bytes32(0)) {
                well = implementation.cloneDeterministic(immutableData, salt);
            } else {
                well = implementation.clone(immutableData);
            }
        } else {
            if (salt != bytes32(0)) {
                well = implementation.cloneDeterministic(salt);
            } else {
                well = implementation.clone();
            }
52:        }
```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L40C9-L52C10

## [NC-05] :Some Contracts are not following proper solidity style guide layout

In some contract variables are declared after function and state changing functions are after view functions which is not proper code layout of solidity.
State variable declaration after functions is not proper code layout of solidity.
_1 Instance_

### Instance#1 :

```solidity
File: src/Well.sol
31:   function init(string memory name, string memory symbol) public initializer {
32:        __ERC20Permit_init(name);
33:        __ERC20_init(name, symbol);

            ...
77:   uint256 constant LOC_AQUIFER_ADDR = 0;
    uint256 constant LOC_TOKENS_COUNT = LOC_AQUIFER_ADDR + PACKED_ADDRESS;
79:    uint256 constant LOC_WELL_FUNCTION_ADDR = LOC_TOKENS_COUNT + ONE_WORD;

```

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L31-L82

Recommendation: Update using below guide layout of variables
Inside each contract, library or interface, use the following order:

Type declarations
State variables
Events
Errors
Modifiers
Functions

https://docs.soliditylang.org/en/latest/style-guide.html#order-of-layout

Order of Functions
Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.

Functions should be grouped according to their visibility and ordered:

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private

https://docs.soliditylang.org/en/latest/style-guide.html#order-of-functions
