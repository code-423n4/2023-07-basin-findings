## [G-01] - consider moving the loop + require check in ``_setReserves`` to a modifier and apply it to all functions changing the reserves, in order to save gas in case of a revert.

## [G-02] - add a ``break`` statement to the loop in ``_getIJ`` if both i and j have been found to avoid looping the entire array.
