
## [G-01] Converting some errors into Custom Errors

https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L40-L63

we can define custom error `error ReserveTooLarge(uint256 reserve);` first and then revert it. 

A total of 5 instances are there.

```
2023-07-basin/blob/main/src/libraries/LibBytes.sol::40 => require(reserves[0] <= type(uint128).max, "ByteStorage: too large");

2023-07-basin/blob/main/src/libraries/LibBytes.sol::41 => require(reserves[1] <= type(uint128).max, "ByteStorage: too large");

2023-07-basin/blob/main/src/libraries/LibBytes.sol::49 => require(reserves[2 * i] <= type(uint128).max, "ByteStorage: too large");

2023-07-basin/blob/main/src/libraries/LibBytes.sol::50 => require(reserves[2 * i + 1] <= type(uint128).max, "ByteStorage: too large");

2023-07-basin/blob/main/src/libraries/LibBytes.sol::63 => require(reserves[reserves.length - 1] <= type(uint128).max, "ByteStorage: too large");
```