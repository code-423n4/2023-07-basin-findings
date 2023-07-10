 Prevent division by 0
On several locations in the code precautions are not being taken for not dividing by 0, this will revert the code. These functions can be called with 0 value in the input, this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero.

https://github.com/code-423n4/2023-07-basin/blob/main/src/functions/ConstantProduct2.sol#L92-L100
````
    function calcReserveAtRatioLiquidity(
        uint256[] calldata reserves,
        uint256 j,
        uint256[] calldata ratios,
        bytes calldata
    ) external pure override returns (uint256 reserve) {
        uint256 i = j == 1 ? 0 : 1;
        reserve = reserves[i] * ratios[j] / ratios[i];
    }
````