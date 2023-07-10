## 1. Directly write `0` instead of `n` in `LibLastReserveBytes.sol/readLastReserves(bytes32)`, this will save gas of one `MLOAD` operation.
https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol#L80

