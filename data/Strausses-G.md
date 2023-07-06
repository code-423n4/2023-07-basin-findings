https://github.com/code-423n4/2023-07-basin/blob/c1b72d4e372a6246e0efbd57b47fb4cbb5d77062/src/pumps/MultiFlowPump.sol#L54

In the constructor, all checks for invalid inputs better be done in the beginning to save more gas in case of fail tx (revert) 

Fix:

constructor(bytes16 _maxPercentIncrease, bytes16 _maxPercentDecrease, uint256 _blockTime, bytes16 _alpha) {
if (_maxPercentDecrease > ABDKMathQuad.ONE) {
            revert InvalidMaxPercentDecreaseArgument(_maxPercentDecrease);
        }
if (_alpha > ABDKMathQuad.ONE) {
            revert InvalidAArgument(_alpha);
        }
...rest of the code
}