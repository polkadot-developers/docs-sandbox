# Build
Type: reference 

## Introduction to languages and tools

Written in the Rust programming language, Substrate comes with powerful out-of-the-box utilities.
These include using `cargo` for documentation, testing and managing dependencies, as well as leveraging the portability and adaptability provided by Web Assembly.
This section outlines reference documentation about the Rust libraries used to build with Substrate and existing libraries written in other programming languages.

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

[ use: https://docs.substrate.io/v3/advanced/scale-codec/]
[ this section would mention some FRAME utilities, not intended to go into all of our FRAME pallets]

`codec`

`scale_info` 

`frame_system`

`frame_support`

`frame_benchmarking`

### Other client libraries

A number of different client libraries designed to interact with Substrate blockchains exist.

[ insert content from: https://docs.substrate.io/v3/integration ]
## Rust basics and best practices

Here is a list of common patterns for using `cargo` when developing with Substrate:

- Generating source code documentation: using `cargo doc` for any pallet or runtime.
- Running unit tests: using `cargo test` for any runtime logic. 
- Updating and adding project dependencies: using `cargo update` and `cargo add`.


Other best practices include:

- using `saturated_add` or `checked_add` for arithmetic operations.
- error checking 

## Why Rust ?

[**Cargo and crates.io**](https://doc.rust-lang.org/book/ch14-00-more-about-cargo.html)
[**Rust blog post**](https://thenewstack.io/rust-by-the-numbers-the-rust-programming-language-in-2021/)
[**Why Rust for smart contracts?**](https://paritytech.github.io/ink-docs/why-rust-for-smart-contracts)

Beyond the fact that Rust provides high performance, type safety and memory efficiency, it provides Substrate with unique characteristics. Generally speaking, these are:

- [**Cargo**](https://doc.rust-lang.org/cargo/guide/why-cargo-exists.html). This is Rust's package management tool, also containing CLI tools for running tests, building documentation, benchmarks and more. 

- [**Crates.io**](https://crates.io/). This is Rust's community managed package registry. 
Any Rust developer can publish their crates there for others to use in their projects. 
This is useful to make Substrate accessible to developers and for developers to reuse existing modules in their projects.

- [**Types, traits and generics**](https://doc.rust-lang.org/book/ch10-00-generics.html). Rust has a sophisticated [trait system](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html) that helps developers make use of Substrate's many layers of abstraction.

- [**Metaprogramming**](https://doc.rust-lang.org/book/ch19-06-macros.html). Substrate uses macros to introduce the concept of "metaprogramming" to its codebase, i.e. writing code that writes code.
For example, [FRAME uses macros](https://docs.substrate.io/v3/runtime/macros/) to alleviate the need to write the heavy lifting code that is required for a pallet to integrate into a runtime. 
Similarly, [ink!](https://paritytech.github.io/ink-docs/) uses macros to handle common type creations and functions.

- [**Webassembly**](https://webassembly.org/). Rust compiles to executable Wasm (Webassembly) byte code, enabling Substrate runtimes to also compile to Wasm. 
Beyond beign a powerful "next-generation web" technology, for Substrate, having Wasm at the core of its design means a few very specific things:
    - **Upgradability**. First and foremost, Wasm is a key piece of technology for [forkless runtime upgrades](https://docs.substrate.io/v3/runtime/upgrades/).
    - **Portability**. Wasm has use cases in selecting validators in cross chain consensus between relay-chain and parachains.
    - **Smart contract compatibility**. Any Smart Contract that compiles to Wasm can be executed by a compatible Substrate node. See a [list of key advantages of having Wasm smart contracts](https://paritytech.github.io/ink-docs/why-webassembly-for-smart-contracts).
    - **Light-client ready**. Wasm is also a key piece in how all Substrate chains are [light-client ready out of the box](https://paritytech.github.io/substrate-connect/#wasm-light-clients). 


See this: https://github.com/substrate-developer-hub/substrate-docs/issues/558. 
Also about std vs. no_std: https://github.com/substrate-developer-hub/substrate-docs/issues/531.

## Create custom pallets

Need to write content that links to basic and intermediate how-to guides.
## Front-end development

Need new content on this and link to tutorials(?)
Add substrate connect: https://github.com/substrate-developer-hub/substrate-docs/issues/573.

## Blockchain-specific best practices

* write efficient code
* simple or complex functions
* minimize database reads/writes
* be mindful of constraints (processing time for computation, network bandwidth, storage, memory)

## Substrate-specific syntax / structures (macros?)

Old content: https://docs.substrate.io/v3/runtime/macros/ 

Need to curate examples from how-to guides or exisiting Substrate code.

Examples of good APIs
Examples of using extrinsics
Examples of using storage

## Cryptographic Keys 

Types of keys to launch, recommendations etc.

See: https://github.com/substrate-developer-hub/substrate-docs/issues/539


