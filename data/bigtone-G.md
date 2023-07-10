
## Gas Optimizations

| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | `reserves[i] == 0` doesn't need to check | 1 |

### <a name="GAS-1"></a>[GAS-1] `reserves[i] == 0` doesn't need to check

#### Impact:
It's already checked if reserves[i] is zero in the `update` function before calling `_init()` , so it doesn't need to check it.

#### Vulnerability Detail


```solidity
File: src/pumps/MultiFlowPump.sol:L153
    function _init(bytes32 slot, uint40 lastTimestamp, uint256[] memory reserves) internal {
        uint256 numberOfReserves = reserves.length;
        bytes16[] memory byteReserves = new bytes16[](numberOfReserves);

        // Skip {_capReserve} since we have no prior reference

        for (uint256 i; i < numberOfReserves; ++i) {
            if (reserves[i] == 0) return; // @audit it doesn't need to check
            byteReserves[i] = reserves[i].fromUIntToLog2();
        }
```


#### Recommendation

Recommend removing the line that checks if reserves[i] == 0
