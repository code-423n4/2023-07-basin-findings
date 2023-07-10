[I-1] Chec-Effects-Interaction is not conformed, with `_setReserves` all executed at the end of the function.

There are multiple occurrence where `_setReserves` is used at the end of the execution context, which would set the latest reserve balanceOf to the storage slot. However functions like swap and addLiquidity/removeLiquidity is state mutating, they all does things like `burn` or `mint` before updating this storage slot, which is not the best practice, and potentially an attack vector if future upgrades/uncareful integration leads to cross-function re-entrency.

### Recommendation:
apply `_setReserves` before executing `burn` or `mint`.