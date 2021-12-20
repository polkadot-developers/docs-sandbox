Substrate is made of a multitude of libraries that facilitate building blockchain clients and their runtime logic. 
At a high level any Substrate blockchain contains:

- A networking layer 
- A consensus layer 
- A transaction pool
- An executor
- A database to store state and blocks
- A keystore
- An RPC layer 

The client can be thought of the outer blockchain infrastructure which handles everything outside the scope of on-chain logic.
Everything responsible for handling on-chain logic happens in the runtime.

Substrate comes with an opinionated toolkit for building runtimes in Rust called FRAME.
That said, the runtime can be built in any way, with a language other than Rust, so long as it can communicate to the client using the primitives it implements. 

# Substrate architecture 

Substrate nodes run a Substrate client application.

![Substrate client architecture](../../img/docs/getting-started/substrate-arch.png)

The Substrate client application provides several important blockchain components including the following:

- **Storage**: used to persist the evolving state of a Substrate blockchain
The blockchain network allows participants to reach trustless [consensus](/v3/advanced/consensus) about the state of storage. 
Substrate ships with a simple and highly efficient [key-value storage mechanism](/v3/advanced/storage).
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

Notes / stuff to add:
- On how light clients work: https://github.com/paritytech/substrate/issues/5047#issuecomment-638708536
- No longer include :code and :heap_pages in execution proofs: https://github.com/paritytech/substrate/pull/10419
- what is the architecture of a Substrate node?