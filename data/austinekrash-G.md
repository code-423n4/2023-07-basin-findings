state variables should be cache rather than reading from them 
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L25C1-L25C1
    uint256 constant ONE_WORD = 32; 
    uint256 constant PACKED_ADDRESS = 20;
    uint256 constant ONE_WORD_PLUS_PACKED_ADDRESS = 52; // For gas efficiency purposes
    bytes32 constant RESERVES_STORAGE_SLOT = bytes32(uint256(keccak256("reserves.storage.slot")) - 1);
