* This analysis report is meant to accompany my Gas Optimizations Report submission. All insights will be given through the perspective of optimizing the codebase.*

## Immediate Impressions
This codebase is extremely efficient as it uses immutable storage in lieu of regular storage and therefore  performs a `calldatacopy (3 gas)` instead of an `SLOAD (2100 gas for cold address/100 gas for warm address)` in order to read storage. What is especially interesting is that this method allows `dynamic arrays` to be stored as immutable data in the `Well`'s bytecode.

## Approach
For the reasons above, there are not many `high impact` optimizations that could be had on this codebase. A majority of the optimizations in my report can be considered `low impact`, with some having the potential to provide decent gas savings given a specified scenario. My focus for this contest was to provide small improvements to the codebase that were unique from the findings in the `bot race`.

## Comments
Unlike my usual reports, I also provided optimizations for `view`/`pure` functions that were not explicitly called within state-mutating functions in this codebase. I did so with the assumption that these `view`/`pure` functions can potentially be called within state-mutating functions in contracts that integrate with Basin.

### Time spent:
8 hours