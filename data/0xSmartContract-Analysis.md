# 🛠️ Analysis - Basin Project 
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Documents  | What is the scope and quality of documentation for Users and Administrators? |
|e) |Centralization risks | How was the risk of centralization handled in the project, what could be alternatives? |
|f) |Security Approach of the Project | Audit approach of the Project |
|g) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|h) |Gas Optimization | Gas usage approach of the project and alternative solutions to it |
|i) |Project team | Details about the project team and approach |
|j) |Other recommendations | What is unique? How are the existing patterns used? |
|k) |New insights and learning from this audit | Things learned from the project |

## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-07-basin#scope

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-07-basin#setup)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review|The [Diagram](https://github.com/code-423n4/2023-07-basin#architecture-diagram) explainer provides a high-level overview of the Project system 
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/code-423n4/2023-07-basin#tests)|The Slither output did not indicate a direct security risk to us, but at some points it did give some insight into the project's careful scrutiny. |
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-07-basin#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-07-basin#scope)|All codes are examined one by one from top to bottom|
|7|Project Audit Reports |[Cyfrin Audit Report](https://basin.exchange/cyfrin-basin-audit.pdf) [Halborn Audit Report](https://basin.exchange/halborn-basin-audit.pdf)  |This reports gives us a clue about the topics that should be considered in the current audit|
|8|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|9|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-07-basin#motivation)|The specific focus areas indicated by the project team are the most sensitive and the potential logic-mathematical errors are also focused on.|


## b) Analysis of the code base
### Well.sol 
Well.sol contract is the most important contract of the project, so most of the audit was on this contract.
The code uses a dynamic immutable storage layout. This allows the contract to be gas-efficient when performing reads.
The code uses a number of Solidity libraries to improve security and efficiency. The code has NatSpec  which makes it easy to understand.
Overall, the code is well-written and easy to understand. It implements a number of important features, such as a constant function AMM and a number of functions for swapping tokens between each other.

### MultiFlowPump.sol 
This contract implements a pump that stores a geometric EMA and cumulative geometric SMA for each reserve. The pump is designed for use in Beanstalk with 2 tokens.
Multi-block MEV resistance reserves: The contract uses a geometric EMA to calculate the instantaneous reserve, which is resistant to MEV attacks.
MEV-resistant geometric EMA: The contract uses a geometric EMA to calculate the instantaneous reserve, which is resistant to MEV attacks.
MEV-resistant cumulative geometric: The contract uses a cumulative geometric SMA to calculate the SMA reserve, which is resistant to MEV attacks.

## c) Test analysis
Test Coverage is 95% and 100% is best practice, tests are mainly unit tests, more integration tests should be added.




## d) Documents 
Documents can be increased, especially the increase in the documentation of mathematical models makes the audit easier and explains what is to be done technically better.
https://basin.exchange/basin.pdf
https://basin.exchange/multi-flow-pump.pdf

##  e) Centralization risks 
No centrality risk

##  f)Security Approach of the Project
Successful current security understanding of the project;
1 - First they did the main audit from a reputable auditing organizations like Halborn and Cyfrin  and resolved all the security concerns in the report
2- They manage the 2nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.

What the project should add in the understanding of Security;
1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)
2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

## g) Other Audit Reports and Automated Findings 

**Automated Findings:**
There are many incorrect notifications in Automatic Finding, it also shows the NounsDao project in the links, this part is completely invalid and has not been taken into account.
https://gist.github.com/CloudEllie/53be96a57d19e7442313cad2a12ec11a

**Other Audit Reports :**
Cryfin ve Halborn
https://basin.exchange/cyfrin-basin-audit.pdf
https://basin.exchange/halborn-basin-audit.pdf

Cryfin and Halborn audit reports are quality reports that have been examined in detail. There are good observations about Read Only Re-Entrancy and Mev, the project team has corrected them.


## h) Gas Optimization
The project is generally gas-optimized and inline-assembly is reasonable in terms of readability and auditability.

In particular, the architecture in the Well contract is gas optimized.
  //////////////////// WELL DEFINITION ////////////////////

    /// This Well uses a dynamic immutable storage layout. Immutable storage is
    /// used for gas-efficient reads during Well operation. The Well must be
    /// created by cloning with a pre-encoded byte string containing immutable
    /// data.
    ///
    /// Let n = number of tokens
    ///     m = length of well function data (bytes)
    ///
    /// TYPE        NAME                       LOCATION (CONSTANT)
    /// ==============================================================
    /// address     aquifer()                  0        (LOC_AQUIFER_ADDR)
    /// uint256     numberOfTokens()           20       (LOC_TOKENS_COUNT)
    /// address     wellFunctionAddress()      52       (LOC_WELL_FUNCTION_ADDR)
    /// uint256     wellFunctionDataLength()   72       (LOC_WELL_FUNCTION_DATA_LENGTH)
    /// uint256     numberOfPumps()            104      (LOC_PUMPS_COUNT)
    /// --------------------------------------------------------------
    /// address     token0                     136      (LOC_VARIABLE)
    /// ...
    /// address     tokenN                     136 + (n-1) * 32
    /// --------------------------------------------------------------
    /// byte        wellFunctionData0          136 + n * 32
    /// ...
    /// byte        wellFunctionDataM          136 + n * 32 + m
    /// --------------------------------------------------------------
    /// address     pump1Address               136 + n * 32 + m
    /// uint256     pump1DataLength            136 + n * 32 + m + 20
    /// byte        pump1Data                  136 + n * 32 + m + 52
    /// ...
    /// ==============================================================

##  i) Project team
Information not available

## j) Other recommendations
Logic similar to singleton pattern similar to Uniswapv4 can be used in a project, this is a gas-optimized and innovative solution
  
## k) New insights and learning from this audit 
-As it is known with Uniswapv4, hook features tend to liberate MEV and this is aimed at eliminating the formability deficiency, the future is in these projects, the Basin project is based on eliminating the formability deficiency, in this context, what I learned most from this project is how this formability architecture can be.
-Multi-block MEV resistance reserves , MEV-resistant geometric EMA,
I gained considerable knowledge in MEV-resistant cumulative geometric designs





### Time spent:
8 hours