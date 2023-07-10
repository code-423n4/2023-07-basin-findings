# Approach
Before the contest started I had researched the `ElasticSwaps` contest report.
On the first day I read the contest documentation, looked up `slither` results and begun codebase manual reviewing.
Over the next few days I continued reviewing the codebase, reading previous audit reports, thinking about possible attack vectors and preparing reports.
# Codebase
The codebase consists of different modules which can be combined into a specific AMM pool. The contest scope includes a part of these modules for uniswap2-like pools. There is also an oracle implementation in the scope which should be refreshed every changing of the constant product.
# Findings contextualizing
I have two findings which I consider as High severity. It was noticed by the project team that modularity can be a source of attack vectors. So I focused on exploiting this weakness in the interaction between the `Well` contract and other modules such as `ConstantProduct2` and `MultiFlowPump contracts.  


### Time spent:
24 hours