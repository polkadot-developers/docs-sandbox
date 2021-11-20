---
title: Smart Contract vs runtime
slug: 
version: '3.0'
section: docs
category: 
keywords: 
---

# Smart Contracts vs. Runtime Development

Developing Substrate runtimes and Smart Contracts are two different approaches to building "decentralized applications" using Substrate.

The following topics provide insight on reasons for choosing Smart Contract development over runtime development for your on-chain logic.


### Choosing the right approach

Substrate runtime development and Smart Contracts each provide solutions designed to solve different problems. There is likely some amount of overlap in the kinds of problems each one can solve, but there is also a clear set of problems suited for only one of the two. To give just one example in each category:

- **Runtime Development:** Building a privacy layer on top of transactions in your blockchain.
- **Smart Contract:** Introducing multi-signature wallets over the currency of your blockchain.
- **Use Case Specific:** Building a gaming dApp which may need to build up a community of users (leaning towards a
  Smart Contract), or may need to scale to millions of transactions a day (leaning more towards Runtime
  Development).

### Cost 
Another consideration is the cost associated with building your dApp using each approach. Deploying a contract is a relatively simple and easy process because you take advantage of the existing network. The only costs to you are the fees which you pay to deploy and maintain your contract.

Setting up your own blockchain, on the other hand, incurs the cost of building a community who find value in the service you provide, or the additional costs associated with establishing a private network with the overhead of a Cloud computing-based architecture and general network maintenance.

While it's hard to address every scenario, in general, runtime development is most favorable for applications that require higher degrees of flexibility and adaptability. For example, applications that require the accommodation of different types of users or multiple layers of governance. The table below is meant to help inform your decisions on which approach to use based on different situations.

| Runtime Development | Smart Contract | Use Case Specific |
|---------------------|:---------------|:------------------|
| Privacy Layer  <br>Feeless Token <br>Light-Client Bridge <br> Decentralized Exchange <br>Oracles <br>Stable Coin| Multi-signature Wallet <br> Data services <br> Simple fundraiser | Small scale gaming dApp (contract) <br>Large scale gaming dApp (runtime) <br> Community driven Decentralized Autonomous Organizations (DAO)(contract)<br> Protocol driven Decentralized Autonomous Organizations (DAO)(runtime) <br> Community driven treasury(contract)<br> Protocol driven treasury (runtime)                |

To learn more about developing with Smart Contracts, see [link](link). <br>

To learn more about runtime development, see [link](link).

> **NOTE:** If you are building on Polkadot, you can also [deploy smart contracts on its parachain](https://wiki.polkadot.network/docs/en/build-smart-contracts). See [The Polkadot Wiki](https://wiki.polkadot.network/docs/build-build-with-polkadot#what-is-the-difference-between-building-a-parachain-a-parathread-or-a-smart-contract) for a comparison between developing on a parachain, parathread, and smart contracts.


# Design

* Smart contract platform
* Smart contract pallet
* EVM pallet
* Prototype using smart contracts
* Plan how to compose a runtime
* Design a pallet
* Storage design decisions
* Query and update efficiency
* Ecomonic models