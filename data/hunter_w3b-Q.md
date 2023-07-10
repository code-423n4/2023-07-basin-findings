## [L-01] Consider the case where lpTokenSupply is 0

Consider the case where lpTokenSupply is 0. When lpTokenSupply is 0, it should return 0 directly, because there will be an error of dividing by 0.

### Impact

This would cause the affected functions to revert and as a result can lead to potential loss.

https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ProportionalLPToken2.sol#L15-L24

```solidity
File: functions/ProportionalLPToken2.sol

22        underlyingAmounts[0] = lpTokenAmount * reserves[0] / lpTokenSupply;

23        underlyingAmounts[1] = lpTokenAmount * reserves[1] / lpTokenSupply;
```

## [L-02] Some tokens may revert when zero value transfers are made

In spite of the fact that EIP-20 states that zero-valued transfers must be accepted, some tokens, such as LEND will revert if this is attempted, which may cause transactions that involve other tokens (such as batch operations) to fully revert. Consider skipping the transfer if the amount is zero, which will also save gas.

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L194

```solidity
File:  src/Well.sol

194        fromToken.safeTransferFrom(msg.sender, address(this), amountIn);


211        amountIn = _safeTransferFromFeeOnTransfer(fromToken, msg.sender, amountIn);


303        fromToken.safeTransferFrom(msg.sender, address(this), amountIn);


431                _tokens[i].safeTransferFrom(msg.sender, address(this), tokenAmountsIn[i]);


780        token.safeTransferFrom(from, address(this), amount);

```

## [L-03] Missing Event for initialize

Events help non-contract tools to track changes, and events prevent users from being surprised by changes Issuing event-emit during initialization is a detail that many projects skip

https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L31-L43

```solidity
File: src/Well.sol

31    function init(string memory name, string memory symbol) public initializer {
32        __ERC20Permit_init(name);
33        __ERC20_init(name, symbol);
34
35        IERC20[] memory _tokens = tokens();
36        for (uint256 i; i < _tokens.length - 1; ++i) {
37            for (uint256 j = i + 1; j < _tokens.length; ++j) {
38                if (_tokens[i] == _tokens[j]) {
39                    revert DuplicateTokens(_tokens[i]);
40                }
41            }
42        }
43    }
```

## [L-04] in this `LibContractInfo.sol` contract uses of staticcall it doesn't provide any protection against Reentrancy Guard

The code uses staticcall to execute the functions on the \_contract address. While staticcall is intended for read-only operations, it does not provide full protection against reentrancy attacks.

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol

```solidity
File: libraries/LibContractInfo.sol

16    function getSymbol(address _contract) internal view returns (string memory symbol) {
17        (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("symbol()"));


34    function getName(address _contract) internal view returns (string memory name) {
35        (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("name()"));


52    function getDecimals(address _contract) internal view returns (uint8 decimals) {
53        (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("decimals()"));
```

## [L-05] there is not any error handling when failed by external function call

The code lacks explicit error handling for failed external function calls. When using staticcall to execute functions on the \_contract address, if the call fails, the library falls back to using bitwise operations (shl, shr) to extract the address data. However, if the fallback mechanism also fails, the function does not indicate any error to the caller.

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibContractInfo.sol#L17

```solidity
File: libraries/LibContractInfo.sol

17        (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("symbol()"));

35        (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("name()"));

53        (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("decimals()"));

```