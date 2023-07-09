### Any comments for the judge to contextualize your findings

During an audit I've compared Basin code with uniswap V2, because it has similar idea, and use
similar approach.
Yes, Basin has significant changes, but they have similar idea:

* For example swap functions -- both protocols have similar approach;
* Skim and sync functions -- both protocols implement them in a similar way;
* Main difference is in calculation liquidity and impossibility to conduct inflation attack;
* Moreover here impossible to take a flash loan due to requires to call transferFrom function before
  any swap.

So, during audit I paid special attention to the exchange of tokens, the addition/deletion of
liquidity and compliance with all invariants.

### Approach taken in evaluating the codebase

Manual review

### Architecture recommendations

From my point of view current architecture is well designed, because swap function uses
transferFrom, this will be better for external services and libs to integrate.
<br>
Moreover authors thought about multiple swaps and created shift function.

### Codebase quality analysis

Code is well documented and tested. A lot of controversial points are explained in the comments to
the code.

### Centralization risks

There is no centralization risks because anyone can deploy any Well with any parameters.

### Mechanism review:

Mechanism is similar to Uniswap V2.

* User may swap tokens, add/remove liquidity;
* The protocol provide ability to record swap information use Pumps, use such approach we easily
  might create either `lastSwapPrice` Pump or `TWAP` Pump or any other service;
* To perform operation required tokens the protocol at the first transfer required amount of tokens
  from sender, *this is a bottleneck* -- because it's impossible to take a flash loan, and I suppose
  it's one of the main functions of the protocol that works with money;
* Exists function to sync balances with reserves;
* Exists function to withdraw stuck reserves wia skim function;
* Moreover, exists shift function -- this function is helpfull to implement multiple swaps in single
  transaction (like a router), but if in uniswap this might be done via swap function, here we need
  to implement another one, because current protocol always use transferFrom before do any
  interactions with money.

### Systemic risks

In Systemic risk I can attribute the inability to continue trading if the reserve of one of the
tokens becomes equal to 0, and this is possible, this may lead to the shutdown of the entire
protocol. Agree it is better to be able to trade but with an incredibly bad price than to receive
transaction revert.

### Time spent:
30 hours