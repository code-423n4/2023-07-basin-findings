- I would certainly recommend more extensive documentation of the assembly parts, which took a bit to crack

- It is unclear why they chose to save the reserves in `log2` format in the pumps, also removing the least-significant bits: with very low liquidity there is a significant loss of precision (i.e. if you store a 3 you read back a 2)


### Time spent:
8 hours