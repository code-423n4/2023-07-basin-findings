
# Low Risk and Non-Critical Issues

# [L-01] Minting tokens to the zero address should be avoided (01 Instances)

Address(0) check is missing in mint function, consider applying check to ensure tokens aren’t minted to the zero address.

Link to the code:
https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L441



# [L-02] Missing event for important parameter change (10 Instances)
Important parameter or configuration changes should trigger an event to allow being tracked off-chain.

Link to the code:
1.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L72
2.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L186
3.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L203
4.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L264
5.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L392
6.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L401
7.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L632
8.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes.sol#L37
9.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibBytes16.sol#L19
10.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol#L19


# [L-03] Use a modifier for access control (07 Instances)

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function’s body.

Link to the code:
1.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L233
2.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L284
3.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L369
4.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L83
5.	https://github.com/code-423n4/2023-07-basin/blob/main/src/pumps/MultiFlowPump.sol#L174
6.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol#L22
7.	https://github.com/code-423n4/2023-07-basin/blob/main/src/libraries/LibLastReserveBytes.sol#L38


# [N-01] Empty blocks should be removed (05 Instances)

Avoid using code blocks or use them for some process like emitting events.

Link to the code:
1.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Aquifer.sol#L27
2.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L114
3.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L656
4.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L657
5.	https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L664 https://github.com/code-423n4/2023-07-basin/blob/main/src/Well.sol#L665
