## 1.correct documentation in MultiFlowPump.sol
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/pumps/MultiFlowPump.sol#L190-L196](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/pumps/MultiFlowPump.sol#L190-L196)
```
-     *     `bytes16 blocksPassed`     <- log2(blocks)
+     *     `bytes16 blocksPassed`     <- blocks 
```


## 2. Make a code similar and better understandable
in _capReserve function in MultiFlowPump.sol
there is if statement make both cases like each other
[https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/pumps/MultiFlowPump.sol#L205-L216](https://github.com/code-423n4/2023-07-basin/blob/9403cf973e95ef7219622dbbe2a08396af90b64c/src/pumps/MultiFlowPump.sol#L205-L216)
```
        if (lastReserve.cmp(reserve) == 1) {
            bytes16 minReserve = lastReserve.add(blocksPassed.mul(LOG_MAX_DECREASE));
            // if reserve < minimum reserve, set reserve to minimum reserve
            if (minReserve.cmp(reserve) == 1) reserve = minReserve;
        }
        // Rerserve Increasing or staying the same.
        else {
            bytes16 maxReserve = blocksPassed.mul(LOG_MAX_INCREASE);
            maxReserve = lastReserve.add(maxReserve);
            // If reserve > maximum reserve, set reserve to maximum reserve
            if (reserve.cmp(maxReserve) == 1) reserve = maxReserve;
        }
```
```
         if (lastReserve.cmp(reserve) == 1) {
-            bytes16 minReserve = lastReserve.add(blocksPassed.mul(LOG_MAX_DECREASE));
+            bytes16 minReserve = blocksPassed.mul(LOG_MAX_DECREASE); //@audit qa better coding
+            minReserve = lastReserve.add(minReserve); 
             // if reserve < minimum reserve, set reserve to minimum reserve
             if (minReserve.cmp(reserve) == 1) reserve = minReserve;
         }
```
