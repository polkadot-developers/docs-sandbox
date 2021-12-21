Section: Build
Sub-section: Libraries
Type: reference 
Index: 1

This article goes over the different libraries available for building blockchains with Substrate. 

## Core libraries

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

> _NOTE: Diagram is a rough sketch. Each part is not meant to be interpreted as nested, rather that "primitives" enable communication between the Clients and Runtimes. Need to annotate diagram with below:_

> - **Client**: Libraries that enable the client and networking layer, including consensus and block execution. 
> - **Primitives**: Libraries responsible for communicating between the client and the runtime, creating the transaction pool and building blocks for the block executor.
> - **FRAME**: Libraries to facilitate building runtime logic and encoding and decoding information passing to and from the runtime.

Each of these components are built from Rust libraries that fall under four categories:

- `sc_*`: Substrate client libraries encapsulate the numerous crates for node and client facing infrastructure, including consensus critical infrastructure, P2P networking, RPC APIs and block execution.
For example, [`sc_service`](https://docs.substrate.io/rustdocs/latest/sc_service/index.html) is responsible for building the networking layer for Substrate blockchains, managing the communication between the network, client and transaction pool. 

- `sp_*`: Substrate primitives are libraries to facilitate communication between the client and the runtime. 
For example, [`sp_std`](https://docs.substrate.io/rustdocs/latest/sp_std/index.html) takes useful primitives from Rust's standard library and makes them usable with any code that depends on the runtime.

- `frame_*`: runtime SDK libraries for building use case specific runtime logic and calling to and from a runtime.
For example, [`frame_support`](https://docs.substrate.io/rustdocs/latest/frame_support/index.html) enables developers to easily declare runtime storage items, errors and events and [`frame_system`](https://docs.substrate.io/rustdocs/latest/frame_system/index.html) acts as the base layer for other pallets to interact with other Substrate components.

- `pallet_*`: a single FRAME module, of which exists an [existing collection](/frame-pallets) created for Polkadot and Kusama. 
Other pallet libraries exist such as the [Open Runtime Module Library (ORML)](https://github.com/open-web3-stack/open-runtime-module-library).

## Other libraries

There are a number of language-specific client libraries that can be used to interact with the [Substrate framework](/link-to-architecture-page). 
In general, the capabilities that these libraries expose are implemented on top of the Substrate remote procedure call (RPC) API.
Although it is possible to build an alternative to [FRAME](./link-to-frame) using Substrate primitives, there has not yet been any significant community efforts to do so yet. 

### Rust

Parity maintains [`subxt`](https://github.com/paritytech/subxt), which is a Rust library specifically designed for submitting extrinsics to Substrate blockchains. 

The [the Substrate API Client](https://github.com/scs/substrate-api-client) is another Substrate client library written in Rust that is maintained by Supercomputing Systems; its API is more general-purpose than `subxt`.
### JavaScript

The Polkadot JS team maintains a rich set of tools for interacting with Substrate-based blockchains.
Refer to [the main Polkadot JS page](../polkadot-js) to learn more about that suite of tools.

Parity also maintains [`txwrapper`](https://github.com/paritytech/txwrapper), which is a Javascript
library for offline generation of Substrate transactions.

### Go

[The Go Substrate RPC Client](https://github.com/centrifuge/go-substrate-rpc-client/) (GSRPC), is
maintained by [Centrifuge](https://centrifuge.io/).

### C#

[Polkadot API DotNet](https://github.com/usetech-llc/polkadot_api_dotnet) is a Substrate RPC client
library for .NET developers. 
It is maintained by [Usetech](https://usetech.com/blockchain/).

[SubstrateNetApi](https://github.com/dotmog/SubstrateNetApi) is a .NET Standard API ([nuget](https://www.nuget.org/packages/SubstrateNetApi)) allowing full Substrate integration in Unity3D for gaming development, [starter template project](https://github.com/darkfriend77/Unity3DExample). 
It is maintained by [DOTMog Team](https://www.dotmog.com/).

### C++

[Usetech](https://usetech.com/blockchain/) also maintains [Polkadot API CPP](https://github.com/usetech-llc/polkadot_api_cpp), which is a C++ library for interacting with the Substrate RPC.

### Python

[py-substrate-interface](https://github.com/polkascan/py-substrate-interface) is a Python
library for interacting with the Substrate RPC. It supports a wide range of capabilities and
powers the [Polkascan multi-chain block explorer](https://polkascan.io/). This library is
maintained by [Polkascan Foundation](https://polkascan.org/).
