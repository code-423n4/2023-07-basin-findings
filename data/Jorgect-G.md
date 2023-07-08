# LOW ISSUE REPORT

## [L-01] A well contract can be front run.
 
The aquire contract is a permissionless Well registry and factory so user can deploy his well contract through this contract. The contract is not forcing the init function and the init function doesn´t has some mechanisc that allow just a deployer to call the init function in the well contract allowing other user to init the contract.

```
file:src/Aquifer.sol
function boreWell(
        address implementation,
        bytes calldata immutableData,
        bytes calldata initFunctionCall,
        bytes32 salt
    ) external nonReentrant returns (address well) {
        ...

        if (initFunctionCall.length > 0) {
            (bool success, bytes memory returnData) = well.call(initFunctionCall); //@audit (low ) frony run init 
            if (!success) {
                // Next 5 lines are based on https://ethereum.stackexchange.com/a/83577
                if (returnData.length < 68) revert InitFailed("");
                assembly {
                    returnData := add(returnData, 0x04)
                }
                revert InitFailed(abi.decode(returnData, (string)));
            }
        }

       ...
    }
```
https://github.com/code-423n4/2023-07-basin/blob/c1b72d4e372a6246e0efbd57b47fb4cbb5d77062/src/Aquifer.sol#L34C4-L65C1

The deployer can decide not init the well contract yet passing 0 in initFunctionCall input.

```
file:src/Well.sol
function init(string memory name, string memory symbol) public initializer {
        
        __ERC20Permit_init(name);
        __ERC20_init(name, symbol);
        ...
       
    }
```
https://github.com/code-423n4/2023-07-basin/blob/c1b72d4e372a6246e0efbd57b47fb4cbb5d77062/src/Well.sol#L31C5-L43C6

There is not protection to allow just a deployer to call the init function allowing other user to front run the well contract and put his own name and symbol.

## Recomendation

force a user to init the well contract in the Aquifer contract:

```
function boreWell(
        address implementation,
        bytes calldata immutableData,
        bytes calldata initFunctionCall,
        bytes32 salt
    ) external nonReentrant returns (address well) {
        ...

        if (initFunctionCall.length == 0) {
            revert CUSTOM_ERROR()
        }

       ...
    }
```