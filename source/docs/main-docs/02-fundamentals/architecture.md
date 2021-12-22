<!--
Notes / stuff to add:
- On how light clients work: https://github.com/paritytech/substrate/issues/5047#issuecomment-638708536

Add section on Design assumptions:
- all blockchains must run on "minimum hardware requirements" 
- protocol vs. infrastructure
-->

Substrate is made of a [multitude of core libraries](./link-todo) that facilitate building blockchain clients and their runtime logic. 
At a high level any Substrate blockchain contains:

- A networking component 
- A consensus component 
- A transaction pool
- An executor
- A database to store state and blocks
- A keystore
- An RPC layer 

The client can be thought of as the outer blockchain infrastructure which handles everything outside the scope of on-chain logic.
Everything responsible for handling on-chain logic happens in the runtime.

This communication layer been the on-chain and outer-node worlds is facilitated by a set of primitives designed for implementing various host functions and runtime APIs.

This means that developers can use Substrate to build the infrastructure requirements for any blockchain protocol so long as it adheres to the communication implementation between a runtime and client.
These are:

- Runtime APIs: allows any client to call into a runtime
- Host functions: allows a runtime to interact with a client

A Substrate node can be thought of as video gaming environment, where the console is the client and the games are the runtime.

For all consensus dependant implementations, runtime APIs and host functions must share the same functionality.
Any consensus-breaking change on the host-function must be reflected in the runtime. 
For example, [the Polkadot Protocol](https://github.com/w3f/polkadot-spec/) specifies host functions and runtime APIs in its implementation for block authoring (BABE) and block finalization (GRANDPA).
Any parachain that uses the Polkadot protocol must also have the same runtime APIs and host functions, where any change on Polkadot's runtime APIs or host functions must be reflected in those of the parachain.
Otherwise there is no way for a relay chain and a parachain to reach consensus.

## Storage

Storage layer: 

- Key-value db allows to build merkle tries ontop of it
- In the KV database everything is an `Option`. Either the storage exists or not. 
- Storage items expose metadata 
- Valuequery → allows to default to 0 if there is no Option
- The storage macro does the hashing of the item, encodes and decodes 
- Hashers: balke2_128 or two_x 
- All user balances has their own key. Reading a storage item is O logn.
- Merkle trie makes it inefficient to read and write, but merkle proofs make it super efficient to know what is there (useful for light clients)

## Light clients

In the video gaming analogy, a light client is the equivalent of a cloud-hosted gaming environment.
Instead of playing straight from a console running some game disk, you're able to access the game environment in a more light-weight manner.

## Assumptions

Substrate nodes run a Substrate client application that makes no assumptions about its environment, except:
- Host function (runtime to interact with Client) and runtime API (client calling into the runtime) are the assumptions we make 
- Every Wasm runtime needs to provide a version
- 

Substrate makes it possible to include executable logic that doesn't take part of consensus, such as off-chain workers.
## Client 

![Substrate client architecture](../../img/docs/getting-started/substrate-arch.png)

The Substrate client application provides several important blockchain components including the following:

- **Storage**: used to persist the evolving state of a Substrate blockchain
The blockchain network allows participants to reach trustless [consensus](/v3/advanced/consensus) about the state of storage. 
Substrate ships with a simple and highly efficient [key-value storage mechanism](/v3/advanced/storage).

[ TODO: Elaborate on storage layers, including externalities]

- **Runtime**: the logic that defines how blocks are processed, including state transition logic. 
In Substrate, runtime code is compiled to [Wasm](/v3/getting-started/glossary#webassembly-wasm) and becomes part of the blockchain's storage state. 
This enables one of the defining features of a Substrate-based blockchain:[forkless runtime upgrades](/v3/runtime/upgrades#forkless-runtime-upgrades). 
Substrate clients can also include a "native runtime" that is compiled for the same platform as the client itself (as opposed to Wasm). 
The component of the client that dispatches calls to the runtime is known as the [executor](/v3/advanced/executor), whose role is to select between the native code and interpreted Wasm. 
Although the native runtime may offer a performance advantage, the executor will select to interpret the Wasm runtime if it implements a newer [version](/v3/runtime/upgrades#runtime-versioning).
- **Peer-to-peer network**: the capabilities that allow the client to communicate with other network participants. 
Substrate uses the Rust implementation of the [`libp2p` network stack](https://libp2p.io/) to achieve this.
- **Consensus**: the logic that allows network participants to agree on the state of the blockchain.
Substrate makes it possible to supply custom consensus engines and also ships with several consensus mechanisms that have been built on top of [Web3 Foundation research](https://w3f-research.readthedocs.io/en/latest/index.html).
- **RPC** (remote procedure call): the capabilities that allow blockchain users to interact with the network. 
Substrate provides HTTP and WebSocket RPC servers.
- **Telemetry**: client metrics that are exposed by the embedded [Prometheus](https://prometheus.io/) server.

## Runtime

A fundamental part of this architecture is the Substrate **runtime**.
The runtime for a blockchain defines the business logic that controls how the blockchain behaves. 
In Substrate-based chains, the runtime provides the [state transition function](../../reference/glossary.md#state-transition-function-stf) because the runtime is where you define the storage items that represent the
blockchain [state](../../reference/glossary.md##state) and the functions that allow blockchain users to make
changes to the state.

![substrate-runtime-client.png](../../img/docs/concepts/substrate-runtime-client.png)

The runtime contains the business logic of the chain. 
It defines what transactions are valid and invalid and determines how the chain's state changes in response to transactions. 
The "outer node", everything other than the runtime, does not compile to Wasm, only to native. The outer node is responsible for handling peer discovery, transaction pooling, block and transaction gossiping, consensus, and answering RPC calls from the outside world. 
While performing these tasks, the outer node sometimes needs to query the runtime for information, or provide information to the runtime. 

## Primitives: infrastructure and protocol

As long as the runtime has mutually understood primitives between the client and itself, it can execute its logic. 
These primitives can be broken down into two categories: everything enabling the implementation of a protocol and the protocol itself.

For example, Polkadot specifies a protocol which requires a set of protocol primitives such as the consenus mechanisms. 
In addition, it builds on the primtive types and data structures the runtime uses for its implementation.
This architecture makes it possible for any Substrate runtime to execute provided that it adheres to the protocol and primitives it shares with the client. 

These protocol primitives are expressed as host functions and runtime APIs, or the "host function/runtime api interface".
The interface between the runtime and the outer node (or client) is made up of host-functions and runtime APIs.

### Host functions and runtime APIs

The "host function/runtime api interface" acts as an independant layer from the features it can enable.
A host function is a function that is expressed outside of the runtime but passed in as imports to it. 
Some examples include:

- 
- 

A runtime API is the interface that enables components outside of the runtime to call into the runtime.
Some examples include:

- 
- 

  *Some protocols*    *Transport layer*    *Some client* 
    ├─────────────┤                      ├─────────────┤                         
    │             │                      │             │ 
    │   Runtime   │ <-- Runtime API --   │   Client    │
    │             │ -- Host functions--> │             │ 
    ├─────────────│                      ├─────────────│        
    

The host functions and runtime APIs provide the means to deliver messages being passed between the runtime and the client. 
They can facilitate a number of runtime implementations without needing to be altered.

These provide any blockchain protocol implemented using Substrate with a core transport layer and on-chain primitives.
For example, the implementation of a runtime based on BABE and AURA adheres to its protocol specification without needing to alter the underlying interface and on-chain primitives it relies on.
The interface is merely the transport layer. 

In the example of a consensus protocol such as with a BABE and AURA runtime, the runtime needs to receive and send messages which is does through the transport layer. 
The ability for a runtime to answer to a request relies on the specific protocol primitive that the runtime and client need to commonly understand.

An important difference is that protocol primitives can be updated without having to update the node-as long as these changes don't require the node to modify behavior around consensus.
However, any change on the host interface does require upgrading the node.
### Runtime APIs

A Runtime API facilitates communication between the outer node and the runtime.
In Substrate, the `sp_api` crate provides an interface to implement a runtime API. 
It is designed to give developers the ability to define their own custom runtime APIs using the [`impl_runtime_apis`](/rustdocs/latest/sp_api/macro.impl_runtime_apis.html) macro. 
However, every runtime must implement the [`Core`](/rustdocs/latest/sp_api/trait.Core.html) and [`Metadata`](/rustdocs/latest/sp_api/trait.Metadata.html) runtime APIs. 
In addition to these, a basic Substrate Node has the following runtime APIs implemented:

- [`BlockBuilder`](/rustdocs/latest/sp_block_builder/trait.BlockBuilder.html): Provides the functionality required for building a block.
- [`TaggedTransactionQueue`](/rustdocs/latest/sp_transaction_pool/runtime_api/trait.TaggedTransactionQueue.html): Handles validating transactions in the transaction queue.
- [`OffchainWorkerApi`](/rustdocs/latest/sp_offchain/trait.OffchainWorkerApi.html): Handles [off-chain capabilities](/v3/concepts/off-chain-features).
- [`AuraApi`](/rustdocs/latest/sp_consensus_aura/trait.AuraApi.html): Handles block authorship with [Aura consensus](/v3/advanced/consensus#aura).
- [`SessionKeys`](/rustdocs/latest/sp_session/trait.SessionKeys.html): Generates and decodes [session keys](/v3/concepts/session-keys).
- [`GrandpaApi`](/rustdocs/latest/sp_finality_grandpa/trait.GrandpaApi.html): Integrates the [GRANDPA](/v3/advanced/consensus#grandpa) finality gadget into the runtime.
- [`AccountNonceApi`](/rustdocs/latest/frame_system_rpc_runtime_api/trait.AccountNonceApi.html): Handles querying transaction indices.
- [`TransactionPaymentApi`](/rustdocs/latest/pallet_transaction_payment_rpc_runtime_api/trait.TransactionPaymentApi.html): Handles querying information about transactions.
- [`Benchmark`](/rustdocs/latest/frame_benchmarking/trait.Benchmark.html): Provides a way to [benchmark](/v3/runtime/benchmarking) a FRAME runtime.

In order to provide its defining forkless runtime upgrade capabilities, Substrate runtimes are built as [WebAssembly (Wasm)](/v3/getting-started/glossary#webassembly-wasm) bytecode. 
Substrate also defines the core primitives that the runtime must implement.

## Core primitives

The Substrate framework makes minimal assumptions about what your runtime must provide to the other layers of Substrate. 
But there are a few data types that need to be defined and must fulfill a particular interface in order to work within the Substrate framework.

They are:

- `Hash`: A type which encodes a cryptographic digest of some data. Typically just a 256-bit
  quantity.

- `DigestItem`: A type which must be able to encode one of a number of "hard-wired" alternatives
  relevant to consensus and change-tracking as well as any number of "soft-coded" variants, relevant
  to specific modules within the runtime.

- `Digest`: A series of DigestItems. This encodes all information that is relevant for a
  light-client to have on hand within the block.

- `Extrinsic`: A type to represent a single piece of data external to the blockchain that is
  recognized by the blockchain. This typically involves one or more signatures, and some sort of
  encoded instructions (e.g. for transferring ownership of funds or calling into a smart contract).

- `Header`: A type which is representative (cryptographically or otherwise) of all information
  relevant to a block. It includes the parent hash, the storage root and the extrinsics trie root,
  the digest and a block number.

- `Block`: Essentially just a combination of `Header` and a series of `Extrinsic`s, together with a
  specification of the hashing algorithm to be used.

- `BlockNumber`: A type which encodes the total number of ancestors any valid block has. Typically a
  32-bit quantity.

## FRAME primitives

Substrate comes with an opinionated toolkit for building runtimes in Rust called FRAME.
That said, the runtime can be built in any way, with any language, so long as it can communicate to the client.

The core Substrate codebase ships with [FRAME](/v3/runtime/frame), Parity's system for Substrate runtime development that is used for chains like [Kusama](https://github.com/paritytech/polkadot/blob/master/runtime/kusama/src/lib.rs) and [Polkadot](https://github.com/paritytech/polkadot/blob/master/runtime/polkadot/src/lib.rs). 
FRAME defines additional runtime primitives and provides a framework that makes it easy to construct a runtime by composing modules, called [pallets](/v3/runtime/frame#pallets). 
Each pallet encapsulates domain-specific logic that is expressed as a set of a [storage items](/v3/runtime/storage), [events](/v3/runtime/events-and-errors), [errors](/v3/runtime/events-and-errors#errors), and [dispatchable functions](/v3/getting-started/glossary#dispatch). 
FRAME developers can [create their own pallets](/v3/runtime/frame#pallets) and reuse existing pallets, including [over 50 of those shipped with Substrate](/v3/runtime/frame#prebuilt-pallets).

![Runtime Composition](../../img/docs/concepts/frame-runtime.png)

There are an additional set of primitives that are assumed about a runtime built with the Substrate FRAME. These are:

- `Call`: The dispatch type that can be called via an extrinsic.

- `Origin`: Represents where a call came from. For example, a signed message (a transaction), an
  unsigned message (an inherent extrinsic), or a call from the runtime itself (a root call).

- `Index`: An account index (aka nonce) type. This stores the number of previous transactions
  associated with a sender account.

- `Hashing`: The hashing algorithm being used in the runtime (e.g. Blake2).

- `AccountId`: The type used to identify user accounts in the runtime.

- `Event`: The type used for events emitted by the runtime.

- `Version`: A type which represents the version of the runtime.

Although a lot of core runtime development can be enabled with FRAME and
its related primitives, FRAME is not the only system for developing
Substrate based blockchains.

## Native and Wasm runtimes

Considered an optimization to Substrate, the native runtime is especially useful for development and testing environments.
It is optional in the sense that production chains don't need to rely on native builds.
The Wasm runtime on the other hand is not optional. 
A chain's Wasm binary is embedded in the client at compile time and required for any [chain specification](./todo).
It is possible to skip the Wasm runtime compilation for developement purposes.
However, launching a chain without one will cause the the chain will panic.
Being a core component to Substrate's design, it provides the possibility for on-chain upgradability and relay chain validation.

There are ongoing discussions about removing the native runtime altogether. 
Refer to this open [issue](https://github.com/paritytech/substrate/issues/7288) for more details.

Here are some reasons why using a native runtime could be desired:

- For development and testing, native runtimes have better debugging support, while Wasm runtimes are more difficult to debug.
- Native execution is faster than Wasm execution and more efficient on slower hardware.

However:

- The Wasm runtime is required in all Substrate chains.
- The Wasm runtime is the canonical encoding of the chains' state transition functions, which implies that something that isn't supported by a Wasm runtime won't be supported by the native runtime.
- In production, on-chain upgrades can only be done with Wasm runtimes.


## Next steps

### Learn more

- Learn about the [Substrate FRAME](/v3/runtime/frame).
- Follow a
  [tutorial to develop your first Substrate chain](/tutorials/v3/create-your-first-substrate-chain).
- Follow a [tutorial to add a pallet to your Substrate runtime](/tutorials/v3/add-a-pallet).

### References

- See the
  [primitive types defined in `node-primitives`](/rustdocs/latest/node_primitives/index.html).

- See the
  [`traits` defined in `sp-runtime`](/rustdocs/latest/sp_runtime/traits/index.html).
