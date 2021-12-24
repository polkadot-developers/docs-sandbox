# Blockchain basics

Blockchain software enables individual computers—called nodes—to communicate with each other to form a decentralized peer-to-peer (P2P) network.
To ensure the security of the data on the chain and the ongoing progress of the chain, the nodes use some form of consensus to agree on the state of data in each block of data and the order in which the blocks are processed.

## What is a blockchain node?

At a high level, a blockchain node consists of the following key components:

- [Storage](/v3/advanced/storage)
- [Peer-to-peer networking](https://libp2p.io)
- [Consensus capabilities](/v3/advanced/consensus)
- Data handling capabilities for external or ["extrinsic"](/v3/concepts/extrinsics) information
- A [Runtime](/v3/concepts/runtime)

Because of the complexity involved in building these components, most blockchain projects are forked from an existing blockchain project.
For example, the Bitcoin repository was forked to create: Litecoin, ZCash, Namecoin, and Bitcoin Cash. Similarly, the Ethereum repository was forked to create Quorum, POA Network, KodakCoin, and Musicoin.

![Blockchain forks](../../img/tutorials/01-create-your-first-chain/forks.png)

However, the existing blockchain platforms were not designed to allow for modification.
As a result, building a new blockchain by forking has serious limitations.

## State machines and handling conflicts

A blockchain runtime is a [state machine](https://en.wikipedia.org/wiki/Finite-state_machine). 
It has some internal state, and state transition function that allows it to transition from its current state to a future state. 
In most runtimes there are states that have valid transitions to multiple future states, but a single transition must be selected.

Blockchains must agree on:

- Some initial state, called "genesis".
- A series of state transitions, each called a "block".
- A final (current) state.

In order to agree on the resulting state after a transition, all operations within a blockchain's [state transition function](/v3/concepts/runtime) must be deterministic.

In centralized systems, the central authority chooses among mutually exclusive alternatives by recording state transitions in the order it sees them, and choosing the first of the competing alternatives when a conflict arises. 
In decentralized systems, the nodes will see transactions in different orders, and thus they must use a more elaborate method to exclude transactions. 
As a further complication, blockchain networks strive to be fault tolerant, which means that they should continue to provide consistent data even if some participants are not following the rules.

Blockchains batch transactions into blocks and have some method to select which participant has the right to submit a block. 
For example, in a proof-of-work chain, the node that finds a valid proof of work first has the right to submit a block to the chain.

## What is Substrate?

Substrate is an open source, modular, and extensible framework for building blockchains.

Substrate is designed to be flexible and allow innovators to design and build a blockchain network that meets their needs.
It provides all the core components you need to build a customized blockchain node.

## Where to go next

Explore the following resources to learn more.

#### Tell me (read related topics)

* [Fundamentals](./index.md)
* 
* 

#### Guide me (related tutorials)

* [Build a local blockchain](../../tutorials/01-build-local-blockchain.md)
* [Simulate a two-node network](../../tutorials/02-simulate-network.md)
* [Start a private network](../../tutorials/03-private-network.md)

#### Show me (related video content)

* 
* 
* 

#### Teach me (how to)

* 
* 
* 

If you prefer to explore code directly, you can start building in the Developer Playground and consult the API reference to get details about the Rust crates you use.
