Section: Design
Sub-section: Runtime development
Type: conceptual 
Index: 1

_This artice aims to describe the concepts around developing Substrate runtimes._
_This should be the entry point for anyone developing with substrate, whereby this content conceptually des ribes the ways one can create application specific business logic for a substrate chain. At a high level, this includes desgning new pallets or smart contracts or using both._

Application logic in Substrate-based blockchains can be expressed in the form of:

- Specialized [pallets](/todo)
- Smart contracts
- A combination of both pallets and smart contracts

Note that designing a Substrate-based blockchain network and designing the application logic of a Substrate-based blockchain are separate things.
While the network type may have an influence on how a runtime is designed and/or how smart contracts are used, the chosen network architecture can evolve over time throughout early phases of development.
For chains already in production, evolving network architecture is possible only with chains with [forkless upgradability](/todo-link) built in, which the Substrate framework provides out of the box.
Refer to [this page](./02-fundamentals/network-types) to explore the types of networks that can be built with Substrate. 

## Smart contracts in Substrate
In order to use smart contracts in a Substrate node, the runtime must be specially configured.
Smart contract compatible runtimes use specialized pallets that provide the type of execution environment the application requires. 
This could be the [EVM pallet](/pallet-todo-link), for example, used in Ethereum compatible Substrate-based chains.
Another example is the [Contracts pallet](/pallet-todo-link) which provides a way to execute Wasm contracts written in a specialized language called ink!. 
Other community projects aiming to provide easy ways to integrate smart contract capabilities also exist.
Refer to this section of the awesome Subtrate repository to learn about them. 

In all cases, the architecture of a Substrate node with smart contract capabilities will look like this:

(TODO: Diagramify)

- Any node will have some pallet (or collection of pallets) that will implement the smart contract execution environment 
- This could be any VM, for example the EVM or a Wasm execution environemnt
- The contract code itself depends on the target environment of the pallet, for example ink! for Contracts pallet

# Smart contracts vs. pallet development
Developing Substrate runtimes and Smart Contracts are two different approaches to building "decentralized applications" using Substrate.

The following topics provide insight on reasons for choosing Smart Contract development over runtime development for your on-chain logic.

## Choosing the right approach
Substrate runtime development and Smart Contracts each provide solutions designed to solve different problems. 
There is likely some amount of overlap in the kinds of problems each one can solve, but there is also a clear set of problems suited for only one of the two. 
To give just one example in each category:

- **Runtime development:** Building a privacy layer on top of transactions in your blockchain.
- **Smart contract:** Introducing multi-signature wallets over the currency of your blockchain.
- **Use case specific:** Building a gaming dApp which may need to build up a community of users (smart contract), or may need to scale to millions of transactions a day (runtime development).

## Cost 
Another consideration is the cost associated with building your dApp using each approach. Deploying a contract is a relatively simple and easy process because you take advantage of the existing network. The only costs to you are the fees which you pay to deploy and maintain your contract.

Setting up your own blockchain, on the other hand, incurs the cost of building a community who find value in the service you provide, or the additional costs associated with establishing a private network with the overhead of a Cloud computing-based architecture and general network maintenance.

While it's hard to address every scenario, in general, runtime development is most favorable for applications that require higher degrees of flexibility and adaptability. For example, applications that require the accommodation of different types of users or multiple layers of governance. The table below is meant to help inform your decisions on which approach to use based on different situations.

| Runtime Development | Smart Contract | Use Case Specific |
|---------------------|:---------------|:------------------|
| Privacy Layer  <br>Feeless Token <br>Light-Client Bridge <br> Decentralized Exchange <br>Oracles <br>Stable Coin| Multi-signature Wallet <br> Data services <br> Simple fundraiser | Small scale gaming dApp (contract) <br>Large scale gaming dApp (runtime) <br> Community driven Decentralized Autonomous Organizations (DAO)(contract)<br> Protocol driven Decentralized Autonomous Organizations (DAO)(runtime) <br> Community driven treasury(contract)<br> Protocol driven treasury (runtime)                |

To learn more about developing runtimes with Smart Contracts, see [link](link). <br>

> **NOTE:** If you are building on Polkadot, you can also [deploy smart contracts on its parachain](https://wiki.polkadot.network/docs/en/build-smart-contracts). See [The Polkadot Wiki](https://wiki.polkadot.network/docs/build-build-with-polkadot#what-is-the-difference-between-building-a-parachain-a-parathread-or-a-smart-contract) for a comparison between developing on a parachain, parathread, and smart contracts.

## Runtime development
Substrate runtime development has the intention of producing lean, performant, and fast nodes. 

You have full control of the underlying logic that each node on your network will run. You have full access to each and every storage item across all of your pallets, which you can modify and control.

With this level of control, you can even brick your chain with incorrect logic or poor error handling. Runtime engineers have much more responsibility for writing robust and correct code, than those deploying Smart Contracts.

When you develop using Substrate runtime, you must provide the protections for the overhead of transaction reverting and implicitly introduce any fee system to the computation of the nodes your chain runs on. This means while you are developing runtime functions, you must correctly assess and apply fees to different parts of your runtime logic so that it will not be abused by malicious actors.

**Substrate Runtime Development:**

- Provides low level access to your entire blockchain.
- Removes the overhead of built-in safety for performance,
  giving developers increased flexibility at the cost of increased responsibility.
- Raises the entry bar for developers, where developers are
  not only responsible for writing working code but must constantly check to avoid writing broken code.
- Has no inherent economic incentives to repel bad actors.

For a comparison of Smart Contract and runtime development, see [link](link).


# Smart contracts development
This article gives an overview of the different ways to implement smart contracts for Substrate-based blockchains.

A traditional smart contract platform allows users to publish additional logic on top of some core blockchain logic. Because smart contract logic can be published by anyone, including malicious actors and inexperienced developers, there are a number of intentional safe guards built around these public smart contract platform.

Some examples are:

- **Fees**: Ensuring that contract execution incurs fees for the computation and storage it forces on the computers running it, so it can't abuse block creators.

- **Sandbox**: A contract is not able to modify core blockchain storage or the storage of other contracts directly. It's power is limited to only modifying it's own state, and the ability to make outside calls to other contracts or runtime functions.

- **Storage Deposit**: A contract takes up space on the blockchain, and should be charged for taking up space on the hard drives of the nodes. This ensures that people don't take advantage of "free, unlimited storage."

- **Reversion**: A contract can be prone to logical errors. The expectations of a contract developer are low, so extra overhead is added to support reverting transactions when they fail, so no state is updated when things go wrong.

These different overheads makes running contracts slower and more costly, but attractive to certain developers.

Contracts allow your community to extend and develop on top of your runtime logic without needing to go through all of the proposals, runtime upgrades, etc... . It can even be used
as a testing ground for future runtime changes, but done in a way that isolates your network from any of the growing pains or errors which might occur.

**Substrate Smart Contracts:**

- Are inherently safer to the network.
- Have built in economic incentives against abuse.
- Have computational overhead to support graceful failures in logic.
- Have a lower bar to entry for development.
- Enable fast-pace community interaction through a playground to write new logic.

For a comparison of Smart Contract and runtime development, see [link](link).
## Prototyping with smart contracts

Todo

