## [L-01] Initializers are not disabled in the `Well` implementation

### Description

Even though the `Well` contract is not upgradable, it leverages the OpenZeppelin upgradable contracts implementations for cloning functionality.

It is recommended to call the `_disableInitializers()` function in the `Well` implementation constructor to prevent any user from calling the `init()` function.

There is no direct risk associated with calling the `init()` function on the `Well` implementation by any user other than the possibility of setting implementation with stupid names for the `name` and `symbol` variables, which can affect the project's reputation.

### Recommendation

Add a constructor with a call to the `_disableInitializers()` function to the `Well` contract.

```diff
    bytes32 constant RESERVES_STORAGE_SLOT = bytes32(uint256(keccak256("reserves.storage.slot")) - 1);
+
+   constructor() {
+       _disableInitializers();
+   }
+
    function init(string memory name, string memory symbol) public initializer {
}
```

## [L-02] Misuse of `shl` function leads to unnecessary over-estimation of size in `calldatacopy` within `_getArgBytes()` function

### Description

In the `ClonePlus` contract, within the `_getArgBytes()` function, the size parameter for `calldatacopy` is incorrectly calculated due to the misuse of the `shl(5, bytesLen)` operation. This operation results in shifting `bytesLen` 5 places to the left, effectively multiplying `bytesLen` by 32 (2^5), leading to an over-estimation of the required size.

While this issue does not have a direct impact on the `data` returned by `_getArgBytes()` — since `data` length is correctly allocated with `bytesLen`, and any excess during the `calldatacopy` call is omitted—it can potentially lead to confusion and unexpected behaviors.

[/src/utils/ClonePlus.sol#L36-L38](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/utils/ClonePlus.sol#L36-L38)

```solidity
File: ClonePlus.sol

36:  assembly {
37:     calldatacopy(add(data, ONE_WORD), offset, shl(5, bytesLen))
38:  }
```
### Recommendation

Avoid unnecessary left-shifting by directly using `bytesLen` as the size parameter in the `calldatacopy` function, which accurately reflects the size of the bytes data to be copied.

```diff
function _getArgBytes(uint256 argOffset, uint256 bytesLen) internal pure returns (bytes memory data) {
    if (bytesLen == 0) return data;
    uint256 offset = _getImmutableArgsOffset() + argOffset;
    data = new bytes(bytesLen);

    // solhint-disable-next-line no-inline-assembly
    assembly {
-       calldatacopy(add(data, ONE_WORD), offset, shl(5, bytesLen))
+       calldatacopy(add(data, ONE_WORD), offset, bytesLen)
    }
}
```

## [N-01] Redundant `SafeCast` usage

### Description

The `SafeCast` library is imported, but its functionality is not used in the `Aquifer`, `Well` and `MultiFlowPump` contracts.

[/src/Well.sol#L8](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L8)
[/src/Well.sol#L23](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L23)

```solidity
File: Well.sol

8:  import {SafeCast} from "oz/utils/math/SafeCast.sol";

23: using SafeCast for uint256;
```

[/src/Aquifer.sol#L6](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Aquifer.sol#L6)
[/src/Aquifer.sol#L20](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Aquifer.sol#L20)

```solidity
File: Aquifer.sol

6:  import {SafeCast} from "oz/utils/math/SafeCast.sol";

20: using SafeCast for uint256;
```

[/src/pumps/MultiFlowPump.sol#L13](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/pumps/MultiFlowPump.sol#L13)
[/src/pumps/MultiFlowPump.sol#L30](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/pumps/MultiFlowPump.sol#L30)

```solidity
File: MultiFlowPump.sol

13:  import {SafeCast} from "oz/utils/math/SafeCast.sol";

30: using SafeCast for uint256;
```

### Recommendation

Remove redundant `SafeCast` library usage.

## [N-02] Redundant imports

### Description

There are redundant import declarations across contracts.

In the `Aquifer` contract the `IWellFunction`, `Well`, `Call` and `IERC20` imports are redundant.

[/src/Aquifer.sol#L8](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Aquifer.sol#L8)
[/src/Aquifer.sol#L10](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Aquifer.sol#L10)

```solidity
File: Aquifer.sol

8:  import {IWellFunction} from "src/interfaces/IWellFunction.sol";

10: import {Well, IWell, Call, IERC20} from "src/Well.sol";
```

### Recommendation

Remove redundant imports.

## [N-03] Not explicit visibility of storage and constant variables

### Description

The code heavily uses storage and constant variables with implicit visibility; this does not directly affect security, but does compromise the code's readability.

[/src/Aquifer.sol#L25](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Aquifer.sol#L25)

```solidity
File: Aquifer.sol

25:  mapping(address => address) wellImplementations;
```

[/src/Well.sol#L26-L29](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L26-L29)
[/src/Well.sol#L77-L82](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L77-L82)

```solidity
File: Well.sol

26: uint256 constant ONE_WORD = 32;
27: uint256 constant PACKED_ADDRESS = 20;
28: uint256 constant ONE_WORD_PLUS_PACKED_ADDRESS = 52; // For gas efficiency purposes
29: bytes32 constant RESERVES_STORAGE_SLOT = bytes32(uint256(keccak256("reserves.storage.slot")) - 1);

77: uint256 constant LOC_AQUIFER_ADDR = 0;
78: uint256 constant LOC_TOKENS_COUNT = LOC_AQUIFER_ADDR + PACKED_ADDRESS;
79: uint256 constant LOC_WELL_FUNCTION_ADDR = LOC_TOKENS_COUNT + ONE_WORD;
80: uint256 constant LOC_WELL_FUNCTION_DATA_LENGTH = LOC_WELL_FUNCTION_ADDR + PACKED_ADDRESS;
81: uint256 constant LOC_PUMPS_COUNT = LOC_WELL_FUNCTION_DATA_LENGTH + ONE_WORD;
82: uint256 constant LOC_VARIABLE = LOC_PUMPS_COUNT + ONE_WORD;
```

[/src/pumps/MultiFlowPump.sol#L36-L39](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/pumps/MultiFlowPump.sol#L36-L39)

```solidity
File: MultiFlowPump.sol

36:  bytes16 immutable LOG_MAX_INCREASE;
37:  bytes16 immutable LOG_MAX_DECREASE;
38:  bytes16 immutable ALPHA;
39:  uint256 immutable BLOCK_TIME;
```

### Recommendation

To improve the code readability, it is recommended to explicitly define the visibility for all storage and constant variables.

## [N-04] Redundant constructor in `Aquifer` contract

### Description

The constructor declaration in the `Aquifer` contract is redundant and decreases code readability.

[/src/Aquifer.sol#L27](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Aquifer.sol#L27)

```solidity
File: Aquifer.sol

27:  constructor() ReentrancyGuard() {}
```

### Recommendation

Remove redundant constructor declaration.

## [N-05] Import from `draft-ERC20PermitUpgradeable.sol` draft contract

### Description

The `ERC20PermitUpgradeable` is final as of 2022-11-01 and should be imported correctly.

[/src/Well.sol#L6](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L6)

```solidity
File: Well.sol

6:  import {ERC20Upgradeable, ERC20PermitUpgradeable} from "ozu/token/ERC20/extensions/draft-ERC20PermitUpgradeable.sol";
```

### Recommendation

Import from `"ozu/token/ERC20/extensions/ERC20PermitUpgradeable.sol";`

## [N-06] Function names are shadowed by parameter names

### Description

In the `init()` function, parameters `name` and `symbol` are shadowing `ERC20Upgradeable.name()` and `ERC20Upgradeable.symbol()` view functions.

[/src/Well.sol#L31-L33](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L31-L33)

```solidity
File: Well.sol

31: function init(string memory name, string memory symbol) public initializer {
32:     __ERC20Permit_init(name);
33:     __ERC20_init(name, symbol);
```

### Recommendation

To avoid any potential confusion or error due to function shadowing, consider renaming the `name` and `symbol` parameters.

## [N-07] `__ReentrancyGuard_init()` call missed in the `Well.init()` function

### Description

The `__ReentrancyGuard_init()` is not called during the `Well.init()` function execution.

In this case there is no direct issue with the missed `__ReentrancyGuard_init()` call, only the first call to any function with `nonReentrant` modifier will use more gas as the storage variable `ReentrancyGuardUpgradeable._status` will be initialized for the first time.

[/src/Well.sol#L31](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/Well.sol#L31)

```solidity
File: Well.sol

31: function init(string memory name, string memory symbol) public initializer {
```

### Recommendation

Add the missing `__ReentrancyGuard_init()` call to the `Well.init()` function to ensure the reentrancy guard is properly set up, reducing the gas cost for subsequent `nonReentrant` function calls.