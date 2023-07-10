## [low] firstPump should check pump exists first

firstPump can be directly called, so it should check if the pump exists. According to https://github.com/code-423n4/2023-07-basin#documentation, including a Pump is optional.

## [low] array length checks in removeLiquidityImbalanced, getRemoveLiquidityImbalancedIn, _addLiquidity, getAddLiquidityOut

```solidity
    function _addLiquidity(
        uint256[] memory tokenAmountsIn,
        uint256 minLpAmountOut,
        address recipient,
        bool feeOnTransfer
    ) internal returns (uint256 lpAmountOut) {
        IERC20[] memory _tokens = tokens();
        uint256[] memory reserves = _updatePumps(_tokens.length);
        // @audit-issue low should check tokenAmountsIn.length == _tokens.length, also in getAddLiquidityOut
```

```solidity
    function removeLiquidity(
        uint256 lpAmountIn,
        uint256[] calldata minTokenAmountsOut,
        address recipient,
        uint256 deadline
    ) external nonReentrant expire(deadline) returns (uint256[] memory tokenAmountsOut) {
        IERC20[] memory _tokens = tokens();
        uint256[] memory reserves = _updatePumps(_tokens.length);
        uint256 lpTokenSupply = totalSupply();

        tokenAmountsOut = new uint256[](_tokens.length);
        _burn(msg.sender, lpAmountIn);
        tokenAmountsOut = _calcLPTokenUnderlying(wellFunction(), lpAmountIn, reserves, lpTokenSupply);
        // @audit-issue low should check minTokenAmountsOut.length == _tokens.length
```

```solidity
    function removeLiquidityImbalanced(
        uint256 maxLpAmountIn,
        uint256[] calldata tokenAmountsOut,
        address recipient,
        uint256 deadline
    ) external nonReentrant expire(deadline) returns (uint256 lpAmountIn) {
        IERC20[] memory _tokens = tokens();
        uint256[] memory reserves = _updatePumps(_tokens.length);
        // @audit-issue low check tokenAmountsOut.length == tokenAmountsOut.length, also in getRemoveLiquidityImbalancedIn

```

## [non-critical] sync should add assert to make sure reserves[i] <= _tokens[i].balanceOf(address(this))

```solidity
    function sync() external nonReentrant {
        IERC20[] memory _tokens = tokens();
        uint256[] memory reserves = new uint256[](_tokens.length);
        for (uint256 i; i < _tokens.length; ++i) {
            reserves[i] = _tokens[i].balanceOf(address(this));
            // @audit-issue non-critical should add assert that reserves[i] <= _tokens[i].balanceOf(address(this))
        }
```

## [non-critical] skim should emit an event when success transfer excess tokens

```solidity
    function skim(address recipient) external nonReentrant returns (uint256[] memory skimAmounts) {
        IERC20[] memory _tokens = tokens();
        uint256[] memory reserves = _getReserves(_tokens.length);
        skimAmounts = new uint256[](_tokens.length);
        for (uint256 i; i < _tokens.length; ++i) {
            skimAmounts[i] = _tokens[i].balanceOf(address(this)) - reserves[i];
            if (skimAmounts[i] > 0) {
                _tokens[i].safeTransfer(recipient, skimAmounts[i]);
            }
        }
        // @audit-issue non-critical should add skim event
```