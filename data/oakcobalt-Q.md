## No event emitting in MultiFlowPump.sol for off-chain accounting

MultFlowPump.sol is designed to be minimalistic in a way that only necessary data is stored on-chain. So it’s an intended behavior some data can be easily acquired and stored off-chain for customized use cases.

For example, for the `TwaReserves` feature to be sufficient, some off-chain accounting might be needed to store historic `CumulativeReserves` values and their timestamps. 

However, the current implementation of MultiFlowPump.sol doesn’t provide such easy access for off-chain mechanisms because there are no events that can be emitted.

### Recommendation

It’s recommended to properly emit events of change in these calculated reserve values and their timestamps for proper off-chain accounting. 