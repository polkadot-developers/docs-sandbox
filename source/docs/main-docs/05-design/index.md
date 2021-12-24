---
title: Design
slug: /docs/05-design
version: '3.0'
section: docs
category: developers
keywords:
---
# Design

A Substrate node can be thought of as video gaming environment, where the console is the client (or the "outer part") and the current game being played is the current runtime (i.e. everything "on-chain").
Each of these components are created using Substrate's [multitude of core libraries](./link-todo) for building blockchain clients and their runtime logic. 

### Runtime APIs and host functions

Any blockchain protocol can be implemented with Substrate by implementing relevant runtime APIs host functions.

[ _TODO: diagram to show how multiple protocols can be implemented with the same runtime api / host function interface_ ]

    |*some protocols*|*transport layer*| *some client* 

    ├─────────────┤                      ├─────────────┤                         
    │             │                      │             │ 
    │   Runtime   │ <-- Runtime API --   │   Client    │
    │             │ -- Host functions--> │             │ 
    ├─────────────│                      ├─────────────│        
    

In the example of a consensus protocol such as with a BABE and AURA runtime, the runtime needs to receive and send messages which is does through the transport layer. 
The ability for a runtime to actually answer to a request relies on the specific protocol primitive that the runtime and client need to commonly understand, which would correspond to some custom host function and runtime API interface. 

An important difference between changes in the runtime API versus the host function interface is that protocol primitives can be updated without having to update the node-as long as these changes don't require the node to modify behavior around consensus.
However, any change on the host interface requires upgrading the node.

Substrate provides developers with the ability to define their own custom runtime APIs using the [`impl_runtime_apis`](/rustdocs/latest/sp_api/macro.impl_runtime_apis.html) macro. 
However, every runtime must implement the [`Core`](/rustdocs/latest/sp_api/trait.Core.html) and [`Metadata`](/rustdocs/latest/sp_api/trait.Metadata.html) runtime APIs. 
In addition to these, the Substrate node template has the following runtime APIs implemented:

- [`BlockBuilder`](/rustdocs/latest/sp_block_builder/trait.BlockBuilder.html): Provides the functionality required for building a block.
- [`TaggedTransactionQueue`](/rustdocs/latest/sp_transaction_pool/runtime_api/trait.TaggedTransactionQueue.html): Handles validating transactions in the transaction queue.
- [`OffchainWorkerApi`](/rustdocs/latest/sp_offchain/trait.OffchainWorkerApi.html): Handles [off-chain capabilities](/v3/concepts/off-chain-features).
- [`AuraApi`](/rustdocs/latest/sp_consensus_aura/trait.AuraApi.html): Handles block authorship with [Aura consensus](/v3/advanced/consensus#aura).
- [`SessionKeys`](/rustdocs/latest/sp_session/trait.SessionKeys.html): Generates and decodes [session keys](/v3/concepts/session-keys).
- [`GrandpaApi`](/rustdocs/latest/sp_finality_grandpa/trait.GrandpaApi.html): Integrates the [GRANDPA](/v3/advanced/consensus#grandpa) finality gadget into the runtime.
- [`AccountNonceApi`](/rustdocs/latest/frame_system_rpc_runtime_api/trait.AccountNonceApi.html): Handles querying transaction indices.
- [`TransactionPaymentApi`](/rustdocs/latest/pallet_transaction_payment_rpc_runtime_api/trait.TransactionPaymentApi.html): Handles querying information about transactions.
- [`Benchmark`](/rustdocs/latest/frame_benchmarking/trait.Benchmark.html): Provides a way to [benchmark](/v3/runtime/benchmarking) a FRAME runtime.

### Coordination with the Runtime

The simplest static consensus algorithms work entirely outside of the runtime as we've described so far. 
However many consensus games are made much more powerful by adding features that require coordination with the runtime. 
Examples include adjustable difficulty in proof of work, authority rotation in proof of authority, and stake-based weighting in proof-of-stake networks.

To accommodate these consensus features, Substrate has the concept of a [`DigestItem`](/rustdocs/latest/sp_runtime/enum.DigestItem.html), a message passed from the outer part of the node, where consensus lives, to the runtime, or vice versa.

[ _TODO: not sure where to put the below, but it related to runtime execution of consenus, and where consensus sits in the big picture_]

A Substrate runtime executes the protocol logic and rules in a deterministic way, such that validators and block producers outside of the runtime can accept or reject them.

In the collator model of a relay chain like Polkadot, collators selection is tightly coupled to the consensus mechanism. 
[ TODO: provide more insight and examples ]

* Smart contract platform
* Smart contract pallet
* EVM pallet
* Prototype using smart contracts
* Plan how to compose a runtime
* Design a pallet
* Storage design decisions
* Query and update efficiency
* Ecomonic models