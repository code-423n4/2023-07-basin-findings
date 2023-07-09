### GAS-1 Using unchecked blocks to save gas - Increments in for loop can be unchecked ( save 30-40 gas per loop iteration)

It is common to utilize for loops that involve incrementing a uint256 variable that is initialized to 0. As the maximum value of uint256 is extremely large, it is unlikely that the variable will ever reach this value before running out of gas. Therefore, it is unnecessary to check for over/underflow during the increment operation. However, the default over/underflow check that is performed in every iteration of almost every for loop can result in wasteful gas consumption.

e.g Let’s work with a sample loop below.
```solidity
for (uint256 i; i < _pumps.length; i++) {
//doSomething
}
```
can be written as shown below.

```solidity
for(uint256 i; i < _pumps.length;) {
  // loop logic
  unchecked { i++; }
}
```

We can also write it as an inlined function like below.

```solidity
function inc(i) internal pure returns (uint256) {
  unchecked { return i + 1; }
}
for(uint256 i; i < _pumps.length; i = inc(i)) {
  // doSomething
}
```

Affected line of code
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L101

```solidity
        uint256 pumpDataLength;
        for (uint256 i; i < _pumps.length; i++) {
            _pumps[i].target = _getArgAddress(dataLoc);
            dataLoc += PACKED_ADDRESS;
            pumpDataLength = _getArgUint256(dataLoc);
            dataLoc += ONE_WORD;
            _pumps[i].data = _getArgBytes(dataLoc, pumpDataLength);
            dataLoc += pumpDataLength;
        }
    }
```

The above should be modified to:

```solidity 
        uint256 pumpDataLength;
-       for (uint256 i; i < _pumps.length; i++) {
+       for (uint256 i; i < _pumps.length) {
            _pumps[i].target = _getArgAddress(dataLoc);
            dataLoc += PACKED_ADDRESS;
            pumpDataLength = _getArgUint256(dataLoc);
            dataLoc += ONE_WORD;
            _pumps[i].data = _getArgBytes(dataLoc, pumpDataLength);
            dataLoc += pumpDataLength;
+      unchecked {
+        ++i;
+      }
        }
    }
```

