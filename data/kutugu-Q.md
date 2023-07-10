# Findings Summary

| ID     | Title                                                                 | Severity |
| ------ | --------------------------------------------------------------------- | -------- |
| [L-01] | LibContractInfo fail to convert the address to string                 | Low      |
| [L-02] | LibContractInfo data decode may fail with wrong type                  | Low      |
| [L-03] | It is possible to create the same well, resulting in less liquidity   | Low      |
| [L-04] | Some slot data is incorrectly overwritten during storage              | Low      |

# Detailed Findings

# [L-01] LibContractInfo fail to convert the address to string

## Description

String is represented by ascii. Digits cannot be directly converted to string; otherwise, garbled characters may occur.

```solidity
    function testSymbol() external returns (string memory symbol) {
        symbol = new string(4);
        address addr = address(0x480b9a9E9Aa1c9db991C7721a92d351Db4FaC990);
        assembly {
            mstore(add(symbol, 0x20), shl(224, shr(128, addr)))
        }
    }
```

```shell
[PASS] testSymbol():(string) (gas: 693)
Traces:
  [693] InvariantTest::testSymbol()
    └─ ← H
          ��
```

## Recommendations

Use openzeppelin Strings library

# [L-02] LibContractInfo data decode may fail with wrong type

## Description

LibContractInfo makes compatible with unsupported interfaces, but not perfect. When function selectors conflict or do not support types, the returned data may not be the correct type. In this case, calling `abi.decode` will revert.

```solidity
    function testWrongTypeRevert() external returns (string memory symbol) {
        symbol = abi.decode(abi.encode(address(1)), (string));
    }
```

## Recommendations

Use the try catch package abi.decode

# [L-03] It is possible to create the same well, resulting in less liquidity

## Description

When creating a well, there is no way to restrict the creation of the same well.     
Even `create2` will create different well if the tokens of immutableData entered by the user are not in the same order or salt is different.      
These same Wells dilute liquidity, and when users swap in the well, they will suffer greater slippage, which will bring capital losses to users.      

## Recommendations

Consider adding a way to limit the creation of duplicate Wells and aggregate liquidity

# [L-04] Some slot data is incorrectly overwritten during storage

## Description

```solidity
// LibBytes16.sol#L45
assembly {
    sstore(
        add(slot, mul(maxI, 32)),
        add(mload(add(reserves, add(iByte, 32))), shr(128, sload(add(slot, maxI))))
    )
}

// LibLastReserveBytes.sol#L58
assembly {
    sstore(
        add(slot, mul(maxI, 32)),
        add(mload(add(reserves, add(iByte, 32))), shr(128, shl(128, sload(add(slot, maxI)))))
    )
}
```

There slot should be overwritten according to the before data, so the last line should read `sload(add(slot, mul(maxI, 32)))` instead of `sload(add(slot, maxI))`.

## Recommendations

Use correct slot data to overwrite
