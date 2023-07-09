# **GAS OPTIMIZATIONS**

## [G-01] Unused Imports
Some files were imported and were not used these files costs gas during deployment and generally this is bad coding practice. You should remove these statements or use the imported files.
### 3 Instances
- `IWellFunction` was imported in the file below with the statement `import {IWellFunction} from "src/interfaces/IWellFunction.sol";` but was not used.
 https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L8

- `Well, Call, IERC20` were imported in the file below in the statement `import {Well, IWell, Call, IERC20} from "src/Well.sol";` but were not used.
https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L10

- `ERC20Upgradeable` was imported in the file below in the statement `import {ERC20Upgradeable, ERC20PermitUpgradeable} from "ozu/token/ERC20/extensions/draft-ERC20PermitUpgradeable.sol";` but were not used.
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L6


## [G-02]   Move lesser gas costing require checks to the top
Revert() statements that check input arguments or cost lesser gas should be at the top of the function.
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.

- https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L307-#L328

```
function readTwaReserves(
        address well,
        bytes calldata startCumulativeReserves,
        uint256 startTimestamp,
        bytes memory
    ) public view returns (uint256[] memory twaReserves, bytes memory cumulativeReserves) {
        bytes16[] memory byteCumulativeReserves = _readCumulativeReserves(well);
        bytes16[] memory byteStartCumulativeReserves = abi.decode(startCumulativeReserves, (bytes16[]));
        twaReserves = new uint256[](byteCumulativeReserves.length);

        // Overflow is desired on `startTimestamp`, so SafeCast is not used.
        bytes16 deltaTimestamp = _getDeltaTimestamp(uint40(startTimestamp)).fromUInt();
        if (deltaTimestamp == bytes16(0)) {
            revert NoTimePassed();
        }
        for (uint256 i; i < byteCumulativeReserves.length; ++i) {
            // Currently, there is no support for overflow.
            twaReserves[i] =
                (byteCumulativeReserves[i].sub(byteStartCumulativeReserves[i])).div(deltaTimestamp).pow_2ToUInt();
        }
        cumulativeReserves = abi.encode(byteCumulativeReserves);
    }
```
In the function above the statements ` bytes16 deltaTimestamp = _getDeltaTimestamp(uint40(startTimestamp)).fromUInt(); ` and ` if (deltaTimestamp == bytes16(0)) {revert NoTimePassed();}` should be moved to the top of the function so that in scenarios where the if condition results to true the function can fail early and cheaply rather than executing gas consuming statements before failing.

The function could be refactored to: 
```
function readTwaReserves(
        address well,
        bytes calldata startCumulativeReserves,
        uint256 startTimestamp,
        bytes memory
    ) public view returns (uint256[] memory twaReserves, bytes memory cumulativeReserves) {
        // Overflow is desired on `startTimestamp`, so SafeCast is not used.
        bytes16 deltaTimestamp = _getDeltaTimestamp(uint40(startTimestamp)).fromUInt();
        if (deltaTimestamp == bytes16(0)) {
            revert NoTimePassed();
        }

        bytes16[] memory byteCumulativeReserves = _readCumulativeReserves(well);
        bytes16[] memory byteStartCumulativeReserves = abi.decode(startCumulativeReserves, (bytes16[]));
        twaReserves = new uint256[](byteCumulativeReserves.length);

        for (uint256 i; i < byteCumulativeReserves.length; ++i) {
            // Currently, there is no support for overflow.
            twaReserves[i] =
                (byteCumulativeReserves[i].sub(byteStartCumulativeReserves[i])).div(deltaTimestamp).pow_2ToUInt();
        }
        cumulativeReserves = abi.encode(byteCumulativeReserves);
    }
```

- https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L285-#L305
```
function _readCumulativeReserves(address well) internal view returns (bytes16[] memory cumulativeReserves) {
        bytes32 slot = _getSlotForAddress(well);
        uint256[] memory reserves = IWell(well).getReserves();
        (uint8 numberOfReserves, uint40 lastTimestamp, bytes16[] memory lastReserves) = slot.readLastReserves();
        if (numberOfReserves == 0) {
            revert NotInitialized();
        }
        uint256 offset = _getSlotsOffset(numberOfReserves) << 1;
        assembly {
            slot := add(slot, offset)
        }
        cumulativeReserves = slot.readBytes16(numberOfReserves);
        uint256 deltaTimestamp = _getDeltaTimestamp(lastTimestamp);
        bytes16 deltaTimestampBytes = deltaTimestamp.fromUInt();
        bytes16 blocksPassed = (deltaTimestamp / BLOCK_TIME).fromUInt();
        // Currently, there is so support for overflow.
        for (uint256 i; i < cumulativeReserves.length; ++i) {
            lastReserves[i] = _capReserve(lastReserves[i], reserves[i].fromUIntToLog2(), blocksPassed);
            cumulativeReserves[i] = cumulativeReserves[i].add(lastReserves[i].mul(deltaTimestampBytes));
        }
    }
```
In the function above it would be better if the `uint256[] memory reserves = IWell(well).getReserves();` comes after the if statement block so that in scenarios where the if condition results to true the EVM would not consume gas for executing the `uint256[] memory reserves = IWell(well).getReserves();` statement.

The code could be refactored to:
```
function _readCumulativeReserves(address well) internal view returns (bytes16[] memory cumulativeReserves) {
        bytes32 slot = _getSlotForAddress(well);
        (uint8 numberOfReserves, uint40 lastTimestamp, bytes16[] memory lastReserves) = slot.readLastReserves();
        if (numberOfReserves == 0) {
            revert NotInitialized();
        }
        uint256[] memory reserves = IWell(well).getReserves();
        uint256 offset = _getSlotsOffset(numberOfReserves) << 1;
        assembly {
            slot := add(slot, offset)
        }
        cumulativeReserves = slot.readBytes16(numberOfReserves);
        uint256 deltaTimestamp = _getDeltaTimestamp(lastTimestamp);
        bytes16 deltaTimestampBytes = deltaTimestamp.fromUInt();
        bytes16 blocksPassed = (deltaTimestamp / BLOCK_TIME).fromUInt();
        // Currently, there is so support for overflow.
        for (uint256 i; i < cumulativeReserves.length; ++i) {
            lastReserves[i] = _capReserve(lastReserves[i], reserves[i].fromUIntToLog2(), blocksPassed);
            cumulativeReserves[i] = cumulativeReserves[i].add(lastReserves[i].mul(deltaTimestampBytes));
        }
    }

```
- https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L239-#L260
```
function readInstantaneousReserves(address well, bytes memory) public view returns (uint256[] memory emaReserves) {
        bytes32 slot = _getSlotForAddress(well);
        uint256[] memory reserves = IWell(well).getReserves();
        (uint8 numberOfReserves, uint40 lastTimestamp, bytes16[] memory lastReserves) = slot.readLastReserves();
        if (numberOfReserves == 0) {
            revert NotInitialized();
        }
        uint256 offset = _getSlotsOffset(numberOfReserves);
        assembly {
            slot := add(slot, offset)
        }
        bytes16[] memory lastEmaReserves = slot.readBytes16(numberOfReserves);
        uint256 deltaTimestamp = _getDeltaTimestamp(lastTimestamp);
        bytes16 blocksPassed = (deltaTimestamp / BLOCK_TIME).fromUInt();
        bytes16 alphaN = ALPHA.powu(deltaTimestamp);
        emaReserves = new uint256[](numberOfReserves);
        for (uint256 i; i < numberOfReserves; ++i) {
            lastReserves[i] = _capReserve(lastReserves[i], reserves[i].fromUIntToLog2(), blocksPassed);
            emaReserves[i] =
                lastReserves[i].mul((ABDKMathQuad.ONE.sub(alphaN))).add(lastEmaReserves[i].mul(alphaN)).pow_2ToUInt();
        }
    }
```
In the function above it would be better if the `uint256[] memory reserves = IWell(well).getReserves();` comes after the if statement block so that in scenarios where the if condition results to true the EVM would not consume gas for executing the `uint256[] memory reserves = IWell(well).getReserves();` statement.

The code could be refactored to: 
```
function readInstantaneousReserves(address well, bytes memory) public view returns (uint256[] memory emaReserves) {
        bytes32 slot = _getSlotForAddress(well);
        (uint8 numberOfReserves, uint40 lastTimestamp, bytes16[] memory lastReserves) = slot.readLastReserves();
        if (numberOfReserves == 0) {
            revert NotInitialized();
        }
        uint256[] memory reserves = IWell(well).getReserves();
        uint256 offset = _getSlotsOffset(numberOfReserves);
        assembly {
            slot := add(slot, offset)
        }
        bytes16[] memory lastEmaReserves = slot.readBytes16(numberOfReserves);
        uint256 deltaTimestamp = _getDeltaTimestamp(lastTimestamp);
        bytes16 blocksPassed = (deltaTimestamp / BLOCK_TIME).fromUInt();
        bytes16 alphaN = ALPHA.powu(deltaTimestamp);
        emaReserves = new uint256[](numberOfReserves);
        for (uint256 i; i < numberOfReserves; ++i) {
            lastReserves[i] = _capReserve(lastReserves[i], reserves[i].fromUIntToLog2(), blocksPassed);
            emaReserves[i] =
                lastReserves[i].mul((ABDKMathQuad.ONE.sub(alphaN))).add(lastEmaReserves[i].mul(alphaN)).pow_2ToUInt();
        }
    }
```
## [G-03]  Declaring redundant variables
 Some varibles were defined even though they are used once. Not defining variables can reduce gas cost and contract size

 ### 2 Instances (12 gas saved)
- https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L524
```
function getRemoveLiquidityOneTokenOut(
        uint256 lpAmountIn,
        IERC20 tokenOut
    ) external view returns (uint256 tokenAmountOut) {
        IERC20[] memory _tokens = tokens();
        uint256[] memory reserves = _getReserves(_tokens.length);
        uint256 j = _getJ(_tokens, tokenOut);
        tokenAmountOut = _getRemoveLiquidityOneTokenOut(lpAmountIn, j, reserves);
    }
```
In the function above the `reserves` variable is declared and is only used once. You could invoke the `_getReserves(token.length)` function directly in the `_getRemoveLiquidityOneTokenOut()` function rather than assigning its value to a memory variable which would cost mstore (3 gas) and mload (3 gas) operations.

The code could be refactored to:
```
function getRemoveLiquidityOneTokenOut(
        uint256 lpAmountIn,
        IERC20 tokenOut
    ) external view returns (uint256 tokenAmountOut) {
        IERC20[] memory _tokens = tokens();
        uint256 j = _getJ(_tokens, tokenOut);
        tokenAmountOut = _getRemoveLiquidityOneTokenOut(lpAmountIn, j, _getReserves(_tokens.length));
    }
```
- https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L487
```
function getRemoveLiquidityOut(uint256 lpAmountIn) external view returns (uint256[] memory tokenAmountsOut) {
        IERC20[] memory _tokens = tokens();
        uint256[] memory reserves = _getReserves(_tokens.length);
        uint256 lpTokenSupply = totalSupply();

        tokenAmountsOut = _calcLPTokenUnderlying(wellFunction(), lpAmountIn, reserves, lpTokenSupply);
    }
```
In the function above the `reserves` variable is declared and is only used once. You could invoke the `_getReserves(token.length)` function directly in the `_calcLPTokenUnderlying()` function rather than assigning its value to a memory variable which would cost mstore (3 gas) and mload (3 gas) operations.

The code could be refactored to:
```
function getRemoveLiquidityOut(uint256 lpAmountIn) external view returns (uint256[] memory tokenAmountsOut) {
        IERC20[] memory _tokens = tokens();
        uint256 lpTokenSupply = totalSupply();

        tokenAmountsOut = _calcLPTokenUnderlying(wellFunction(), lpAmountIn, _getReserves(_tokens.length), lpTokenSupply);
    }

```

## [G-04]  Empty blocks should be removed or emit something
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
- https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L656
- https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L664
