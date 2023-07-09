# [L-01]

In Aquifer.sol in function `boreWell` sue structure instead of `bytes calldata immutableData`.
<br>
Better use structured parameter, for better integration, because it's complicated to understand
whole new protocol with assembler code to understand how to create new Well.

Better to create structure like:

```solidity
struct WellInit {
    address [] tokens;
    address [] pumps;
    // ... and so on 
}
```

# [L-02] 

Please, fix Typos:

`// Rerserve Increasing or staying the same.` -> reserve 
<br>
`Multi-block MEV resistence reserves` -> resistance
<br>
`* recieves `s * b_i / S` of each underlying token.` -> receives


