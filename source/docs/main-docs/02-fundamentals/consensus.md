_This article presents reference and conceptual information about the different consensus mechanisms used in Substrate._ 

Consensus is an integral runtime component that implements the rules which defines the changes of state from one block to the next in the chain.

There are different consensus mechanisms with various tradeoffs that are supported "out-of-the-box" in Substrate.

The standard [Substrate node template](https://github.com/substrate-developer-hub/substrate-node-template) uses BABE and AURA.

A Substrate runtime executes the protocol logic and rules in a deterministic way, such that validators and block producers outside of the runtime can accept or reject them.

In the collator model of a relay chain like Polkadot, collators selection is tightly coupled to the consensus mechanism. 
[ TODO: provide more insight and examples ]



