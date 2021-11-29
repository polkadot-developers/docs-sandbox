Section: Build
Type: reference 
Index: 1

## Introduction to languages and tools

Written in the Rust programming language, Substrate comes with powerful out-of-the-box utilities.
These include using `cargo` for documentation, testing and managing dependencies, as well as leveraging the portability and adaptability provided by Web Assembly.
This section outlines the reference documentation about the libraries that are used to build with Substrate as well other tools and libraries written in other programming languages.

## Substrate core libraries 

Substrate's core libraries include all crates necessary to build and run runtimes in a Substrate node.
Although some libraries are cross functional in how they can be used, each of them can be put in the following categories:

- **Client**: Libraries which enable the client and networking layer, including consensus and block execution. 
- **Runtime**: Libraries responsible for calling into a runtime, creating the transaction pool and building blocks for the block executor.
- **FRAME and SCALE Codec**: Libraries to facilitate building runtime logic and encoding and decoding runtime data.

### Client 

These libraries, denoted `sc_` which stands for "Substrate Client", encapsulate the numerous Substrate crates for node and client facing infrastructure, including consensus critical infrastructure, P2P networking, RPC APIs and block execution.

[`sc_service`](https://docs.substrate.io/rustdocs/latest/sc_service/index.html)
    
A crate responsible for building the networking layer for Substrate blockchains, managing the communication between the network, client and transaction pool. 
    
[`sc_network`](https://docs.substrate.io/rustdocs/latest/sc_network/index.html)

`sc_basic_authorship`

`sc_cli`

`sc_client_api`

`sc_consensus`

`sc_consensus_aura`

`sc_executor`

`sc_finality_grandpa`

`sc_keystore`

`sc_rpc`

`sc_rpc_api`

`sc_telemetry`

`sc_transaction_pool`

`sc_transaction_pool_api`

`substrate_frame_rpc_system`

### Runtime 
Libraries in this category are denoted `sp_`, or "Substrate primitives" and do the heavy lifting for building highly customizable runtimes.

[`sp_std`](https://docs.substrate.io/rustdocs/latest/sp_std/index.html): A crate that handles low-level primitive types for Substrate runtimes.
This library takes useful primitives from Rust's standard library and makes them usable with any code that depends on the runtime.

[`sp_api`](https://docs.substrate.io/rustdocs/latest/sp_api/index.html): This is Substrate's runtime API, which exposes traits that help with the creation and customization of APIs designed to make calls to and from the runtime of a Substrate node.

[`sp_io`](https://docs.substrate.io/rustdocs/latest/sp_io/index.html)

`substrate_wasm_builder`: This crate is a tool to configure which Wasm binaries are built and how. 
For example, to skip the Wasm build when building a Substrate node, running `SKIP_WASM_BUILD=1 cargo --build` would create a native build only. 
Read more on various environment variables to configure [here](https://docs.substrate.io/rustdocs/latest/substrate_wasm_builder/index.html#environment-variables).

`sp_block_builder` 

`sp_core`

`sp_inherents`

`sp_offchain`

`sp_runtime`

`sp_session`

`sp_transaction_pool`

`sp_version`

`sp_timestamp`

`sp_blockchain`

`sp_consensus_aura`

### SCALE and FRAME

[ this section isn't meant for FRAME pallets, but the lower level FRAME crates ]

[ use: https://docs.substrate.io/v3/advanced/scale-codec/]

`codec`

`scale_info` 

`frame_system`

`frame_support`

`frame_benchmarking`

### Other client libraries

A number of different client libraries designed to interact with Substrate blockchains exist.

[ insert content from: https://docs.substrate.io/v3/integration ]

## Substrate tools 

Developers building with Substrate can use a number of tools depending on where they are in their development cycle.

[ Paste content from: https://docs.substrate.io/v3/tools/ ]
