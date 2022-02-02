<!--
Notes / stuff to add:
- On how light clients work: https://github.com/paritytech/substrate/issues/5047#issuecomment-638708536

Add section on Design assumptions:
- all blockchains must run on "minimum hardware requirements" 
- protocol vs. infrastructure
- advanced topic: some teams create runtimes without FRAME and pallets, but using FRAME helps to decompose the state transition function nicely.
- advanced callout: You do not see `&self` in FRAME traits so that all state is in storage, and it is difficult to introduce side-effects
-->

A Substrate node is designed to be modular and adaptable to change. 
This article presents the architecture of a Substrate node, using the [node template](https://github.com/substrate-developer-hub/substrate-node-template) as a reference which provides a set of core components ready to use out of the box.
Any of these components can be swapped out for different ones, depending on the target optimization or use case.

![Substrate client architecture](../../img/docs/getting-started/substrate-arch.png)

A Substrate node can be thought of as video gaming environment, where the console is the client (or the "outer part") and the current game being played is the current runtime (i.e. everything "on-chain").
Each of these components are created using Substrate's [multitude of core libraries](/main-docs/06-build/libraries/) for building blockchain clients and their runtime logic. 

The architecture of a Substrate node contains:

- **[A runtime](#runtime)**: the logic that defines how blocks are processed, including state transition logic. 
In Substrate, runtime code is compiled to [Wasm](/v3/getting-started/glossary#webassembly-wasm) and becomes part of the blockchain's storage state. 
This enables [forkless runtime upgrades](/v3/runtime/upgrades#forkless-runtime-upgrades). 
Substrate clients can also include a "native runtime". 
Everything responsible for handling on-chain logic and state persistence happens in the runtime.
- **[A storage component](#storage)**: used to persist the evolving state of a Substrate blockchain.
Substrate ships with a simple and highly efficient [key-value storage mechanism](/v3/advanced/storage).
- **[An executor](#executor)**: the component of the client that dispatches calls to the runtime is known as the [executor](/v3/advanced/executor), whose role is to select between the native code and interpreted Wasm. 
The executor will select to interpret the Wasm runtime if it implements a newer [version](/v3/runtime/upgrades#runtime-versioning).
- **A network layer**: the capabilities that allow the client to communicate with other network participants. 
Substrate uses the Rust implementation of the [`libp2p` network stack](https://libp2p.io/).
- **A consensus engine**: the logic that allows network participants to agree on the state of the blockchain.
Substrate makes it possible to supply custom consensus engines and also ships with several consensus mechanisms that have been built on top of [Web3 Foundation research](https://w3f-research.readthedocs.io/en/latest/index.html).
- **An RPC API**: the capabilities that allow blockchain users to interact with the network. 
Substrate provides HTTP and WebSocket RPC servers.
- **A telemetry layer**: client metrics that are exposed by the embedded [Prometheus](https://prometheus.io/) server.

## Runtime

The runtime contains the business logic of the chain. 
It defines what transactions are valid and invalid and determines how the chain's state changes in response to transactions. 

![substrate-runtime-client.png](../../img/docs/concepts/substrate-runtime-client.png)

The "outer node", everything other than the runtime is responsible for handling peer discovery, transaction pooling, block and transaction gossiping, consensus, and answering RPC calls from the outside world. 
While performing these tasks, the outer node sometimes needs to query the runtime for information, or provide information to the runtime. 

### Runtime APIs and host functions

Any blockchain protocol can be implemented with Substrate by implementing relevant runtime APIs and host functions.
This is possible by defining a [runtime interface](https://docs.substrate.io/rustdocs/latest/sp_runtime_interface/index.html) to serve as a transport layer between a Substrate runtime and client.

[ _TODO: diagram to show how multiple protocols can be implemented with the same runtime api / host function interface_ ]

              |*some protocols*| *transport layer*|   *some client* 

                ├─────────────┤                      ├─────────────┤                         
                │             │                      │             │ 
                │   Runtime   │ <-- Runtime API --   │   Client    │
                │             │ -- Host functions--> │             │ 
                ├─────────────│                      ├─────────────│        
    

Host functions and runtime APIs provide a means to deliver messages being passed between the runtime and the client. 
Substrate can facilitate a number of runtime implementations without needing to alter the host functions and runtime APIs that come out of the box.

A **host function** is a function that is expressed outside of the runtime but passed in as an import to the runtime. 
One example is the benchmarking implementation in FRAME.
[Benchmarking host functions](https://docs.substrate.io/rustdocs/latest/frame_benchmarking/benchmarking/struct.HostFunctions.html) are required to extract benchmarks generated by the hardware running the node and therefore need to make calls into the node.

A **runtime API** facilitates communication between the outer node and the runtime.
Benchmarking requires a [runtime API](https://docs.substrate.io/rustdocs/latest/frame_benchmarking/trait.Benchmark.html) as well, in order for calling into the runtime to perform benchmarking tasks.

In the example of a consensus protocol such as with a BABE and AURA runtime, the runtime needs to receive and send messages which is does through the runtime interface.
The ability for a runtime to actually answer to a request relies on some primitive at the protocol level that the runtime and client need to commonly understand, which would correspond to some custom [host function](https://docs.substrate.io/rustdocs/latest/sp_wasm_interface/trait.HostFunctions.html) and [runtime API](https://docs.substrate.io/rustdocs/latest/sp_api/index.html).

An important difference between making changes in the runtime API versus the host function interface is that protocol primitives, such as runtime parameters, can be updated without having to update the node &mdash; as long as these changes don't require the node to alter some consensus mechanism.
However, any change on the host interface would require upgrading the node.

### Runtime APIs

Substrate provides the following runtime APIs:

- [`Core`](/rustdocs/latest/sp_api/trait.Core.html): A core runtime API that every Substrate runtime must implement. It handles things like runtime versioning and block execution. 
- [`Metadata`](/rustdocs/latest/sp_api/trait.Metadata.html): A runtime API that every Substrate runtime must implement. It returns metadata for the runtime.
- [`BlockBuilder`](/rustdocs/latest/sp_block_builder/trait.BlockBuilder.html): Provides the functionality required for building a block.
- [`TaggedTransactionQueue`](/rustdocs/latest/sp_transaction_pool/runtime_api/trait.TaggedTransactionQueue.html): Handles validating transactions in the transaction queue.
- [`OffchainWorkerApi`](/rustdocs/latest/sp_offchain/trait.OffchainWorkerApi.html): Handles [off-chain capabilities](/v3/concepts/off-chain-features).
- [`AuraApi`](/rustdocs/latest/sp_consensus_aura/trait.AuraApi.html): Handles block authorship with [Aura consensus](/v3/advanced/consensus#aura).
- [`SessionKeys`](/rustdocs/latest/sp_session/trait.SessionKeys.html): Generates and decodes [session keys](/v3/concepts/session-keys).
- [`GrandpaApi`](/rustdocs/latest/sp_finality_grandpa/trait.GrandpaApi.html): Integrates the [GRANDPA](/v3/advanced/consensus#grandpa) finality gadget into the runtime.
- [`AccountNonceApi`](/rustdocs/latest/frame_system_rpc_runtime_api/trait.AccountNonceApi.html): Handles querying transaction indices.
- [`TransactionPaymentApi`](/rustdocs/latest/pallet_transaction_payment_rpc_runtime_api/trait.TransactionPaymentApi.html): Handles querying information about transactions.
- [`Benchmark`](/rustdocs/latest/frame_benchmarking/trait.Benchmark.html): Provides a way to [benchmark](/v3/runtime/benchmarking) a FRAME runtime.

Learn more on how to [design custom "runtime API / host function" interfaces](./link-todo-design) in Substrate.

### Native and Wasm runtimes

Substrate runtimes can be compiled to both [WebAssembly (Wasm)](/v3/getting-started/glossary#webassembly-wasm) and native.

The native runtime is useful for development and certain testing environments. 
Using it, developers don't need to wait so long for building their chain and can use Rust standard library features for debugging and logging.
On the other hand, the Wasm runtime is a `no_std` environemnt which implies that developers need to use its own system for debugging and logging. 

Being a defining feature provided by Substrate, the Wasm runtime is a requirement for any chain in production. 
It provides chains with the ability to perform [forkless runtime upgrades](../09-maintain/upgrade) and is also required for parachains to submit blocks for relay chain validation.
In a production environemnt, the chain's Wasm binary is embedded in the client at compile time, making it an integral part of the [build process](../06-build/build-process) of a Substrate chain.

<!-- 
There are ongoing discussions about removing the native runtime altogether. 
Refer to this open [issue](https://github.com/paritytech/substrate/issues/7288) for more details.

Here are some reasons why using a native runtime could be desired:

- For development and testing, native runtimes have better debugging support, while Wasm runtimes are more difficult to debug.
- Native execution is faster than Wasm execution and more efficient on slower hardware.

However:

- The Wasm runtime is required in all Substrate chains.
- The Wasm runtime is the canonical encoding of the chains' state transition functions, which implies that something that isn't supported by a Wasm runtime won't be supported by the native runtime.
- In production, on-chain upgrades can only be done with Wasm runtimes. -->

## Storage

Substrate uses a simple key-value data store implemented as a database-backed, modified [Merkle tree](https://en.wikipedia.org/wiki/Merkle_tree).
This allows any higher-level storage abstraction to be built ontop of this simple key-value storage layer.
All events, extrinsics and pallet logic use this storage layer to persist state on-chain.
For example, the Wasm runtime of a Substrate chain is stored at the magic key [`:code`](https://docs.substrate.io/rustdocs/latest/sp_storage/well_known_keys/constant.CODE.html).
User balances are stored as a [`StorageMap`](https://docs.substrate.io/rustdocs/latest/frame_support/storage/trait.StorageMap.html) which uses the key-value database.

This layer is built using the Rust implementation of [Rocks DB](http://rocksdb.org/).
There is a different implementation under developement called [Parity DB](https://github.com/paritytech/parity-db), also built in Rust but aims to optimize storage and retrieval of state data. In any case, Substrate is designed to support any key-value database implementation. 

[ TODO: Elaborate on storage layers, including externalities]

**Trie abstraction**

Substrate uses a Base-16 Modified Merkle Patricia tree ("trie") from
[`paritytech/trie`](https://github.com/paritytech/trie) to provide a trie structure whose contents can be modified and whose root hash is recalculated efficiently.

Substrate-based chains have a single main trie, called the state trie, whose root hash is placed in each block header. 
This is used to easily verify the state of the blockchain and provide a basis for light clients to verify proofs.

This trie only stores content for the canonical chain, not forks. 
There is a separate [`state_db` layer](/rustdocs/latest/sc_state_db/index.html) that maintains the trie state with references counted in memory for all non-canonical blocks.
All trie nodes are stored in the database and part of the trie state can get pruned, i.e. a key-value pair can be deleted from storage when it is out of pruning range for non-archive nodes. 

Substrate also provides an API to generate new [child tries](./todo-link-design) with their own root hashes that can be used in the runtime.
Learn about ways to design and implement the storage for your chain [here](./todo-link-design).

## Executor

The [executor](/v3/getting-started/glossary#executor) is responsible for dispatching and executing calls into the Substrate runtime.

The native runtime is included as part of the node executable, while the Wasm binary is stored on the blockchain under a [well known storage key](https://docs.substrate.io/rustdocs/latest/sp_storage/well_known_keys/index.html).
These two representations of the runtime may not be the same. 
After the runtime is upgraded, the executor determines which version of the runtime to use when dispatching calls.

Before runtime execution begins, the Substrate client proposes which runtime execution environment should be used. 
This is controlled by the execution strategy, which can be configured for the different parts of the blockchain execution process. 
The strategies are listed in the [`ExecutionStrategy` enum](/rustdocs/latest/sp_state_machine/enum.ExecutionStrategy.html):

- `NativeWhenPossible`: Execute with native build (if available, WebAssembly otherwise).
- `AlwaysWasm`: Only execute with the WebAssembly build.
- `Both`: Execute with both native (where available) and WebAssembly builds.
- `NativeElseWasm`: Execute with the native build if possible; if it fails, then execute with WebAssembly.

All strategies respect the runtime version, meaning if the native and Wasm runtime versions differ, the Wasm runtime is chosen.
These are configurable using Substrate's [CLI](./link-to-build-cli).

**Wasm execution**

The Wasm representation of the Substrate runtime is considered the canonical runtime. 
Because this Wasm runtime is placed in the blockchain storage, the network must come to consensus about this binary. 
Thus it can be verified to be consistent across all syncing nodes.

The Wasm execution environment can be more restrictive than the native execution environment. 
For example, the Wasm runtime always executes in a 32-bit environment with a configurable memory limit (up to 4 GB).

For these reasons, the blockchain prefers to do block construction with the Wasm runtime. 
Some logic executed in Wasm will always work in the native execution environment, but the same cannot be said the other way around. 
Wasm execution can help to ensure that block producers create valid blocks.

**Native execution**

The native runtime will only be used by the executor when it is chosen as the execution strategy and it is compatible with the requested runtime version (see [Runtime Versioning](/v3/runtime/upgrades#runtime-versioning)).
For all other execution processes other than block construction, the native runtime is preferred since it is more performant. 
In any situation where the native executable should not be run, the canonical Wasm runtime is executed instead.

## Primitives

The Substrate framework makes minimal assumptions about what your runtime must provide to the other layers of Substrate.  
As long as the runtime has primitives that are mutually understood by the client, it can execute its logic. 
These primitives can be broken down into two categories: everything enabling the implementation of a protocol (core primitives) and the protocol itself.

**Core primitives** are the data types that need to be defined and must fulfill a particular interface in order to work within the Substrate framework.

These are:

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

**Protocol primitives** are a more abstract class of primitives. 
They are typically informed by consensus critical components which define the relationship between a client and a runtime.
These relate to implementations of [runtime APIs and host functions](#runtime-apis-and-host-functions), whereby any consensus-breaking change on the host-function must be reflected in the runtime. 

Adhering to some set of protocol primitives is necessary for chains connecting to eachother such as in Polkadot's parachain model.
For example, [the Polkadot protocol](https://github.com/w3f/polkadot-spec/) specifies host functions and runtime APIs in its implementation for block authoring (BABE) and block finalization (GRANDPA).
Any parachain that uses the Polkadot protocol must also expose the same runtime APIs and host functions.
Similarly, any change in Polkadot's runtime APIs or host functions must be reflected in the parachain, otherwise there is no way for a relay chain and a parachain to reach consensus.

This architecture makes it possible to implement any Substrate runtime, using any language or libraries provided that it adheres to the protocol and primitives it shares with the client. 
