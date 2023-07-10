In ConstantProduct2.sol

https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/functions/ConstantProduct2.sol#L58

The lpTokenSupply is redundant as the reserve could be retrieved without a reference to it, along with the roundup function. Only a precision and conditional check could be useful.