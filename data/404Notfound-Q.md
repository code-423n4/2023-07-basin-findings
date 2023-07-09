## Missing validation of the pump length in `firstPump()`
### Proof of Concept
The `firstPump()` function is called by the `_updatePumps()`, which will check carefully if the pump length equals 1.
```solidity
    function _updatePumps(uint256 _numberOfTokens) internal returns (uint256[] memory reserves) {
        reserves = _getReserves(_numberOfTokens);

        if (numberOfPumps() == 0) {
            return reserves;
        }

        // gas optimization: avoid looping if there is only one pump
        if (numberOfPumps() == 1) {
            Call memory _pump = firstPump();
```
However, the `firstPump()` is a public function that can be directly called by other contracts, if the pump length is zero, an invalid pump will be returned.
https://github.com/code-423n4/2023-07-basin/blob/e1b03e74a87954892ff8c32dfd647972ec6e6a8f/src/Well.sol#L172-L178

### Recommended Mitigation Steps
Check if the pump length is over zero.
```
    function firstPump() public pure returns (Call memory _pump) {
        if (numberOfPumps() == 0) return _pumps;
        uint256 dataLoc = LOC_VARIABLE + numberOfTokens() * ONE_WORD + wellFunctionDataLength();
        _pump.target = _getArgAddress(dataLoc);
        uint256 pumpDataLength = _getArgUint256(dataLoc + PACKED_ADDRESS);
        _pump.data = _getArgBytes(dataLoc + ONE_WORD_PLUS_PACKED_ADDRESS, pumpDataLength);
    }
```