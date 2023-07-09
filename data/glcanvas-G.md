# [G-01]

In Well.sol in updatePumps use variable instead of calling numberOfPumps() twice
```solidity
    function _updatePumps(uint256 _numberOfTokens) internal returns (uint256[] memory reserves) {
        reserves = _getReserves(_numberOfTokens);

        if (numberOfPumps() == 0) { // todo create variable
            return reserves;
        }

        // gas optimization: avoid looping if there is only one pump
        if (numberOfPumps() == 1) {
            Call memory _pump = firstPump();
            // Don't revert if the update call fails.
            try IPump(_pump.target).update(reserves, _pump.data) {}
            catch {
                // ignore reversion. If an external shutoff mechanism is added to a Pump, it could be called here.
            }
        } else {
            Call[] memory _pumps = pumps();
            for (uint256 i; i < _pumps.length; ++i) {
                // Don't revert if the update call fails.
                try IPump(_pumps[i].target).update(reserves, _pumps[i].data) {}
                catch {
                    // ignore reversion. If an external shutoff mechanism is added to a Pump, it could be called here.
                }
            }
        }
    }
```