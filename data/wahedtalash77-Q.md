# Low Risk finding

## [1] UNDERSCORES CAN BE ADDED FOR NUMBERS
It is a common practice to separate each 3 digits in a number by an underscore to improve code readability. Unlike ONE_WORD, the following PACKED_ADDRESS, ONE_WORD_PLUS_PACKED_ADDRESS, and RESERVES_STORAGE_SLOT do not use underscores.

```
26:    uint256 constant ONE_WORD = 32;
27:    uint256 constant PACKED_ADDRESS = 20;
28:    uint256 constant ONE_WORD_PLUS_PACKED_ADDRESS = 52; 
29:    bytes32 constant RESERVES_STORAGE_SLOT = bytes32(uint256(keccak256("reserves.storage.slot")) - 1);
```

https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L26C4-L29C103

==Also here==
```
uint256 constant LOC_AQUIFER_ADDR = 0;
    uint256 constant LOC_TOKENS_COUNT = LOC_AQUIFER_ADDR + PACKED_ADDRESS;
    uint256 constant LOC_WELL_FUNCTION_ADDR = LOC_TOKENS_COUNT + ONE_WORD;
    uint256 constant LOC_WELL_FUNCTION_DATA_LENGTH = LOC_WELL_FUNCTION_ADDR + PACKED_ADDRESS;
    uint256 constant LOC_PUMPS_COUNT = LOC_WELL_FUNCTION_DATA_LENGTH + ONE_WORD;
    uint256 constant LOC_VARIABLE = LOC_PUMPS_COUNT + ONE_WORD;
```
https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L77C5-L82C64