Section: Build
Type: reference 
Index: 1

This article goes over the different libraries and tools available for building blockchains with Substrate. 
## Substrate libraries

At a high level, Substrate consists of libraries to build: a client, a runtime and the communication layer between the two. 



                                ┌────────────────────────┐
                                │ Client                 │
                                │                        │
                                │                        │
                                │                        │
                                │     ┌──────────────────┤
                                │     │ Primitives       │
                                │     │                  │
                                │     │     ┌────────────┤
                                │     │     │            │
                                │     │     │            │
                                │     │     │   Runtime  │
                                └─────┴─────┴────────────┘

> _TODO: Need to annotate diagram with below._

> - **Client**: Libraries that enable the client and networking layer, including consensus and block execution. 
> - **Primitives**: Libraries responsible for communicating between the client and the runtime, creating the transaction pool and building blocks for the block executor.
> - **FRAME**: Libraries to facilitate building runtime logic and encoding and decoding information passing to and from the runtime.

Each of these components are built from Rust libraries that fall under four categories:

- `sc_*`: Substrate client libraries encapsulate the numerous crates for node and client facing infrastructure, including consensus critical infrastructure, P2P networking, RPC APIs and block execution.
For example, [`sc_service`](https://docs.substrate.io/rustdocs/latest/sc_service/index.html) is responsible for building the networking layer for Substrate blockchains, managing the communication between the network, client and transaction pool. 

- `sp_*`: Substrate primitives are libraries to facilitate communication between the client and the runtime. 
For example, [`sp_std`](https://docs.substrate.io/rustdocs/latest/sp_std/index.html) takes useful primitives from Rust's standard library and makes them usable with any code that depends on the runtime.

- `frame_*`: runtime SDK libraries for building use case specific runtime logic and calling to and from a runtime.
For example, [`frame_support`](https://docs.substrate.io/rustdocs/latest/frame_support/index.html) enables developers to easily declare runtime storage items, errors and events.

- `pallet_*`: a single FRAME module, of which exists an [existing collection](/frame-pallets) created for Polkadot and Kusama. 
Other pallet libraries exist such as the [Open Runtime Module Library (ORML)](https://github.com/open-web3-stack/open-runtime-module-library).

## Other libraries

Other libraries designed to interact with the [Substrate framework](/link-to-architecture-page) exist, primarily for Substrate clients.

[ inserted content below from: https://docs.substrate.io/v3/integration and https://docs.substrate.io/v3/integration/client-libraries/]

### Polkadot-JS

The [Polkadot-JS project](https://polkadot.js.org/docs/) is a collection of tools, interfaces, and libraries around Polkadot and Substrate.
While the project is named after "Polkadot", these tools, interfaces, and libraries are fully compatible with any Substrate based chain.

#### Polkadot-JS API

The API provides application developers the ability to query a node and interact with the Polkadot or Substrate chains using Javascript.
Go to [documentation](https://polkadot.js.org/docs/api).

The Polkadot-JS API is a [library of interfaces](https://github.com/polkadot-js/api) for communicating with Polkadot and Substrate nodes.

#### Polkadot-JS Apps

The Polkadot-JS Apps is a flexible UI for interacting with a Polkadot or Substrate based node.
Go to [documentation](https://polkadot.js.org/apps).

This is pre-built [user-facing application](https://github.com/polkadot-js/apps), allowing access to all features available on Substrate chains.

To connect the Polkadot-JS Apps to your local node, you must go into `Settings` and change the
"endpoint to connect to" to `Local Node (127.0.0.1:9944)`.

If you are connected to the Polkadot-JS Apps over a secure HTTPS connection, you will need to use a browser which also supports bridging to an insecure WebSocket endpoint. 
For example, Google Chrome supports this, but Mozilla Firefox does not.

#### Polkadot-JS extension

The Polkadot-JS Extension is a simple proof-of-concept for managing accounts in a browser extension and allowing the signing of extrinsics using these accounts. 
It also provides a simple interface for interacting with extension-compliant dApps.

Different ways to use the extension:

- [On Chrome](https://chrome.google.com/webstore/detail/polkadot%7Bjs%7D-extension/mopnmbcafieddcagagdcbnhejhlodfdd)

- [On Firefox](https://addons.mozilla.org/en-US/firefox/addon/polkadot-js-extension)

- [Fork on GitHub](https://github.com/polkadot-js/extension)



Although it is possible to build an alternative to [FRAME](./link-to-frame) using Substrate primitives, there has not yet been any significant community efforts to do so. 
## Substrate tools 

Developers building with Substrate can use a number of tools depending on where they are in their development cycle.

[ Paste content from: https://docs.substrate.io/v3/tools/ ]

### Client 
    
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

### Runtime crates

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

### SCALE and FRAME crates

[ this section isn't meant for FRAME pallets, but the lower level FRAME crates ]

[ use: https://docs.substrate.io/v3/advanced/scale-codec/]

`codec`

`scale_info` 

`frame_system`

`frame_support`

`frame_benchmarking`

