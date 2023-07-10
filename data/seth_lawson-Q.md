# NC-1 : Remove forge-std import

forge-std is used for logging and debugging and should be removed when not being used for development.

script\deploy\Aquifer.s.sol:
  4: import {Script} from "forge-std/Script.sol";

script\deploy\AquiferWell.s.sol:
  5: import {Script, console} from "forge-std/Script.sol";

script\deploy\MockPump.s.sol:
  4: import {Script, console} from "forge-std/Script.sol";

script\deploy\Well.s.sol:
  5: import {console2} from "forge-std/console2.sol";
  6: import {console} from "forge-std/console.sol";
  7: import {Test} from "forge-std/Test.sol";
  8: import {Script} from "forge-std/Script.sol";

script\deploy\helpers\Logger.sol:
  5: import {console2} from "forge-std/console2.sol";
  6: import {console} from "forge-std/console.sol";
