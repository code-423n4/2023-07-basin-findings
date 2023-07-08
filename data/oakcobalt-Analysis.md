### Any comments for the judge to contextualize your findings

The main focus of the audit is to:
1. Ensure core external-facing contracts operate as intended as outlined in the docs.
2. Identify potential security issues, especially opportunities for exploits within the smart contracts.
3. Special focus was paid to the protocol goals of achieving 'composability' in AMMs. Evaluate how the current design of pumps and wells might compromise the purposed 'composability', as well as how they might compromise common essential features of popular AMM protocols like Uniswap as a trade-off.

Note that library contracts were excluded from my audit.

### Approach taken in evaluating the codebase
1. Manual code walkthrough.
2. Manual assessment of variables to identify any arithmetic-related vulnerabilities.
3. Testing with custom scripts. (Foundry)

### Architecture recommendations

Since 'well functions' is the core that enables different flavors of trade, it might be debatable whether a design pattern that completely separates 'well functions' and 'wells' will limit the versatility of AMMs. 

An example that might challenge this pattern is UniswapV3, or similar AMMs that provide concentrated liquidity. In UniswapV3, liquidity is nonfungible and the current frameworks of Well.sol are insufficient in supporting nonfungible liquidity. In addition, in 'well functions', the relations between total supply and token reserve do not exist in this scenario. 

One recommendation is to explore how an ERC721-based well might achieve a clean separation between well and well functions for protocols like UniswapV3. Otherwise, might need to consider a new group of wells with integrated well functions that deal with nonfungible liquidity which compromises 'composability' on some level.

### Codebase quality analysis
1. Documentation: 
The code is largely well documented with some exceptions in Well.sol, MultiFlowPump.sol, ConstantProduct2.sol where public-facing functions have no or insufficient Natspec comments. There are inconsistent Natspec styles in ConstantProduct2.sol.

2. Tests: 
There are uncovered areas, although the extent is not assessed. For example, in Pump.Fuzz.t.sol, there is no testing on EMA values -`emaReserves[i]` when there is testing on the last reserves and cumulative reserves.

3. Gas:
Wasteful gas operation is observed. The report is submitted separately.

### Centralization risks

Very low risk of centralization

### Mechanism review

One concern on the assumptions in Well.sol is the acceptance of uneven liquidity adding or removing. As an example of a good well implementation, Well.sol allows various uneven and single-sided liquidity modifications by users with no constraints. 

Although this is a design choice, it allows token reserve ratios in a market to change outside of normal trading activities, which can be considered a vulnerable feature for an AMM.

Note that as an alternative, single-sided liquidity modification can be achieved by including token swapping, in which case it can be an add-on feature without impacting ratios of token reserves.

### Systemic risks

In my opinion, the pump design is crucial in ensuring overall system health. 
1. The current MultiFlowPump.sol implementation has some weaknesses in handling multiple token prices. 

2. In addition, there seems to be no implementation in place to allow multiple wells to share a pump. For example, the doc says a pump can be shared by multiple wells if it stores a mapping of well addresses. Such implementation is not found.

3. Because pumps are intended to be designed in a minimalistic way, further tests and example implementation of how pumps' core functionality such as EMA, SMA, and TWA values can be extended on-chain and off-chain need to be looked into. 







 






### Time spent:
11 hours