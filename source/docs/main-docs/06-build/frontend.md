Section: Build
Sub-section: Front-end development
Type: reference 

Frontend application development for Substrate runtimes could be for any type of user-facing interface to a Substrate blockchain, such as browser applications, desktop applications or low consumption hardware environments. 

All front-end libraries that interact with Substrate runtimes are made possible by leveraging the rich metadata system provided by [`frame-metadata`](https://docs.substrate.io/rustdocs/latest/frame_metadata/index.html) and [`scale-info`](https://docs.rs/scale-info/latest/scale_info/).


                            ┌─────────────┐   
                            │RPC functions|    
                            └─────────────┘ 
                                 │   ▲
                                 ▼   │        
                            ┌─────────────┐   
                            │Type Registry|  
                            └─────────────┘   
                              scale-info 
                                   ▲
                                   │                                 
                             frame-metadata
                                   ▲
                                   │
                            ┌──────┴────────┐
                            │    Runtime    │
                            │  (n pallets)  │
                            └───────────────┘

                            
- The runtime contains pallets that expose callable functions, types, parameters and documentation for each pallet.
- `frame-metadata` helps decode the metadata of all the types and callable functions the runtime exposes.
- `scale-info` creates a "registry" of all types generated from `frame-metadata`. The types in this registry are annotated so they can be encoded and decoded to the SCALE binary format as well as information about how each type will be encoded.

With >V14 metadata, any runtime can be queried and expose the available runtime calls, types and parameters.  
The metadata also exposes how a type is expected to be decoded, making it easy for an external application to retrieve any of the information a runtime exposes. 

[add content from "c-metadata"]

Different libraries exist for building frontend interfaces of Substrate-based chains and interacting with Substrate runtimes.
## subxt 

`subxt` is a Rust library for submitting extrinsics to a Substrate node.
It leverages the combination of `frame-metadata` and `scale-info` and removes the maintainance burden of updating and recompiling an app's API everytime a node introduces new types.

It provides a way for Rust developers to write user facing client applications for any Substrate chain without the burden of maintaining a list of types. 
Being built with Rust, it's also a good solution for lower level applications such as non-browser graphic user interfaces.

Integrated into the `subxt` library, the [`subxt-cli` tool](./reference/command-line-tools) can be used to generate the metadata as well as the runtime API of a target running chain.
## Polkadot-JS

The [Polkadot-JS project](https://polkadot.js.org/docs/) is a collection of tools, interfaces, and libraries around Polkadot and Substrate.
While the project is named after "Polkadot", these tools, interfaces, and libraries are fully compatible with any Substrate based chain.

The [Polkadot JS API](https://polkadot.js.org/docs/api) provides application developers the ability to query a node and interact with the Polkadot or Substrate chains using Javascript.


## Substrate connect

[Substrate Connect](https://paritytech.github.io/substrate-connect/) is a JavaScript library and browser extension that builds on the [Polkadot JS API](/v3/integration/polkadot-js#polkadot-js-api) to enable developers to build application specific light clients for Substrate chains. 
By using light clients available to all Substrate built blockchains, application developers no longer need to rely on single RPC nodes to allow end-users to interact with their applications. 
This introduces a new paradigm for decentralization: instead of specifying a centralized RPC node, developers just need to specify the blockchain's [chain specification](/v3/runtime/chain-specs) for their application to synchronize with the chain.

Substrate Connect provides application developers ways to run a Substrate light client in any NodeJS environment, from in-browser applications and extensions, to Electron apps, IOT devices, and mobile phones.

For in-browser end-users, Substrate Connect is a browser extension designed to facilitate using applications with multiple blockchains, where all light clients can run in a single tab.

This implies two key features:

1. **Ready-to-use light clients for Substrate chains.** Light clients are part of the Substrate framework and with that, available for every Substrate based blockchain. 
This means that all you need in order to connect a Substrate chain to your application is provide the [chain specification](/v3/runtime/chain-specs) of the chain you want to connect to.

2. **Bundling light-clients of multiple chains.** With the browser extension, end-users are able to interact with applications connected to multiple blockchains or connect their own blockchains to applications that support it.

**Motivation**

Interacting with a Substrate chain via an RPC server requires a layer of third party trust which can be avoided. 
Substrate Connect uses a Wasm light client which connects to a Substrate chain without any unecessary intermediary.

In addition, due to browser limitations on websockets from HTTPS pages, establishing a good number of peers is difficult as nodes are reachable only if they provide a TLS (Transport Layer Security) certificate.
Substrate Connect provides a browser extension to overcome this limitation and to keep the chains synced in the background, making applications on a Substrate chain faster.

**How it works**

The extension runs a single light client, [Smoldot](https://github.com/paritytech/smoldot) that manages connecting to different blockchains. 
Whenever a user opens an app in a new browser tab it asks the extension to connect to whatever blockchains the app is interested in. 
The light client is smart enough to share resources so that it only connects to a network once even if there are multiple apps talking to it.

The [`@substrate/connect`](https://www.npmjs.com/package/@substrate/connect) library has the following capabilities:

- It detects whether a user has the Substrate Connect browser extension installed. 
If it isn't installed, it falls back to instantiating a light client directly in the page.

- It handles receiving and listening for messages from the browser extension and provider.

- It manages an app's connection to multiple blockchains, creating an instance of Smoldot and connecting the app to it.

**Usage**

When used in individual projects, the Substrate Connect node module will first check for the installed extension. 
If available, it will try to connect to the light client running inside the extension. 
Only if the extension is not installed will it start a light client in the browser tab.

Learn how to integrate Substrate Connect in your applications [here](https://paritytech.github.io/substrate-connect/).

[ _TODO : Add substrate connect note: https://github.com/substrate-developer-hub/substrate-docs/issues/573_ ].
