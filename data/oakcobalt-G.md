## Wasteful gas operation is performed during an update to pumps in the same block
When MultiFlowPump.sol `update()` is called, `blockPassed` is calculated based on `deltaTimestamp`. When the function is called during the same block as the last reserve update, `deltaTimestamp` will be 0 and `blockPassed` is 0. 

This is the condition where wasteful gas occurs. 

In MultiFlowPump.sol `update()`, `lastReserves[i]` is designed to be the same value as the previous update with the same timestamp as the last timestamp. `cumulativeReserve[i]` will be the same as the previous `cumulativeReserve[i]`.

```solidity
//MultiFlowPump.sol-update()
...
       {
            uint256 deltaTimestamp = _getDeltaTimestamp(
                pumpState.lastTimestamp
            );
            alphaN = ALPHA.powu(deltaTimestamp);
            deltaTimestampBytes = deltaTimestamp.fromUInt();
            // Relies on the assumption that a block can only occur every `BLOCK_TIME` seconds.
            blocksPassed = (deltaTimestamp / BLOCK_TIME).fromUInt();
        }
...
            pumpState.lastReserves[i] = _capReserve(
                pumpState.lastReserves[i],
                (reserves[i] > 0 ? reserves[i] : 1).fromUIntToLog2(),
                blocksPassed
            );
...
slot.storeLastReserves(uint40(block.timestamp), pumpState.lastReserves);
...
```
```solidity
//MultiFlowPump.sol-_capReserve()
...
        if (lastReserve.cmp(reserve) == 1) {
            bytes16 minReserve = lastReserve.add(
                blocksPassed.mul(LOG_MAX_DECREASE)
            );
            // if reserve < minimum reserve, set reserve to minimum reserve
            if (minReserve.cmp(reserve) == 1) reserve = minReserve;
        }
...
```
Even though `lastReserve[i]`, `cumulativeReserve[i]`, and `lastTimestamp` don’t change, these values will still be calculated inside `_capReserve `and then written to storage. This is a wasteful calculation and storage operation that can be bypassed to save gas.

### Recommendation

Bypass `lastReserves[i]` `cumulativeReserves[i]` calculation and re-writing to storage when `deltaTimestamp` is 0. 



