Do not use full size words for low value variables (more if they are constant). Check [here](https://github.com/code-423n4/2023-07-basin/blob/c1b72d4e372a6246e0efbd57b47fb4cbb5d77062/src/Well.sol#L26-L28)

Change strings in `require` to custom errors, saving gas on the way. The regex pattern is `"\w*"\);` and loop through the occurrences

Use `unchecked { i++; }` in loops where the end is known. The regex pattern is `; ?\w*\++`. Save gas because solidity checks integers for over/underflow after 0.8.0 by default, so we can omit that because we know the limit