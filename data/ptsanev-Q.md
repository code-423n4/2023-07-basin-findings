## [L-01] - both ``skim()`` and ``sync()`` are susceptible to front-running and cancel each other out. A malicious actor can front-run regular callers to either sync or take excess before them and make them waste their gas.

Mitigation: add necessary require checks to avoid calling said functions and wasting gas if nothing would happen. In this case it would be to have a boolean and if any of the reserves update, then set it to true and check it, otherwise avoid the ``_setReserve`` if no changes have occured.

## [L-02] - in the case of a 2 token Well, that uses the default ConstantProduct2 function, the shift function would always revert since ``amountOut = reserves[j] - _calcReserve(wellFunction(), reserves, j, totalSupply());`` with no changes to ``reserves[i]`` would amount to 0 < minAmountOut.

Mitigation: Change the reserves of the 'i' token in the above case inside the function or notify the user either through docs or comment lines that he needs to do so.