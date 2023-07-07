1) LibWellConstructor::encodeWellInitFunctionCall()
   The convention of name and symbol is mixed up in the above function leading to confusion while reading the code. Refer to how symbol read from LibContractInfo is assigned to name.
 
   ```solidity
   function encodeWellInitFunctionCall(
        IERC20[] memory _tokens,
        Call memory _wellFunction
    ) public view returns (bytes memory initFunctionCall) {
        string memory name = LibContractInfo.getSymbol(address(_tokens[0]));
        string memory symbol = name;
        for (uint256 i = 1; i < _tokens.length; ++i) {
            name = string.concat(name, ":", LibContractInfo.getSymbol(address(_tokens[i])));
            symbol = string.concat(symbol, LibContractInfo.getSymbol(address(_tokens[i])));
        }
        name = string.concat(name, " ", LibContractInfo.getName(_wellFunction.target), " Well");
        symbol = string.concat(symbol, LibContractInfo.getSymbol(_wellFunction.target), "w");

        // See {Well.init}.
        initFunctionCall = abi.encodeWithSignature("init(string,string)", name, symbol);
    } 

    ```