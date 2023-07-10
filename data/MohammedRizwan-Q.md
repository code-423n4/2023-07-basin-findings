## Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | draft-ERC20PermitUpgradeable is deprecated by openzeppelin | 1 |
| [L&#x2011;02] | Violation of Checks, Effect and Interactions pattern in Well.sol | 1 |
| [L&#x2011;03] | Use latest version of openzeppelin library | 1 |

### [L&#x2011;01]  draft-ERC20PermitUpgradeable is deprecated by openzeppelin
Openzeppelin has deprecated the use of draft-ERC20PermitUpgradeable in v4.9.0 release. Check out the deprecations [here](https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.9.0).

There is 1 instance of this issue:

### Recommended Mitigation steps
Use [ERC20PermitUpgradeable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC20/extensions/ERC20PermitUpgradeable.sol).

### [L&#x2011;02]  Violation of Checks, Effect and Interactions pattern
It is always recommended to follow Checks, Effect and Interactions(CEI) pattern in contracts to prevent re-entrancy attacks.

There is 1 instance of this issue:

In Well.sol at [L-371](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L369-L371).

For example:

```Solidity

        if (amountOut >= minAmountOut) {
-           tokenOut.safeTransfer(recipient, amountOut);
            reserves[j] -= amountOut;
+           tokenOut.safeTransfer(recipient, amountOut);

        // Some code
```

### [L&#x2011;03]  Use latest version of openzeppelin library
The contracts has used old version of OZ library. Library contracts security should be upto date. Use the latest version of OZ library i.e v4.9.2.