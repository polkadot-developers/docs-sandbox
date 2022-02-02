Section: Build
Sub-section: Front-end development
Type: reference 

Application development against Substrate runtimes encompasses building user-facing interfaces such as browser and desktop applications, as well as client applications for specific hardware requirements, such as hardware wallets.
Different libraries exist to build these types of applications, depending on your needs.
This article explains the process of querying a Substrate node and using the metadata it exposes in order to provide background knowledge to help inform these decisions, or to help developers create application specific libraries.

> NOTE: Although it's technically possible to use any protocol to communicate with a Substrate node, this article assumes that the chosen interface is via a node's RPC interface using JSON-RPC.

## Metadata system 

Substrate nodes provide an RPC call, `state_getMetadata`, that returns a complete description of all the types in the current runtime. Client applications use the metadata to interact with the node, to parse responses and format message payloads sent to the node.
This includes information about a pallet's storage items, transactions, events, errors and constants.
The current version (V14) differs significantly from its predecessors as it allows clients to generate information of the types used in the runtime.
This means that if a runtime containts a pallet with some custom type, the type information will be included as part of the metadata returned.
This rich metadata system is made possible by both the [`frame-metadata`](https://docs.substrate.io/rustdocs/latest/frame_metadata/index.html) and [`scale-info`](https://docs.rs/scale-info/latest/scale_info/) crates which all Substrate runtimes using the latest system have.

In order to actually send and receive transactions to and from the node, front-end APIs must implement a [SCALE codec library](./libraries#SCALE-Codec) to encode and decode RPC payloads.

The general flow of how metadata is generated, exposed and used to make and receive calls from the runtime:

- Callable pallet functions, as well as types, parameters and documentation are exposed by the runtime.
- The `frame-metadata` crate describes the structure in which the information about how to communicate with the runtime will be provided. 
The information takes the form of a type registry provided by `scale-info`, as well as information about things like which pallets exist (and what the relevant types in the registry are for each pallet).
- The `scale-info` crate is used to annotate types across the runtime, and makes it possible to build a registry of runtime types. This type information is detailed enough that we can use it to find out how to correctly SCALE encode or decode some value for a given type.
- The structure described in `frame-metadata` is populated with information from the runtime, and this is then SCALE encoded and made available via the `state_getMetadata` RPC call.
- Custom RPC APIs use the metadata interface and provide methods to make calls into the runtime. 
A SCALE codec library is required to encode and decode calls to and from these RPCs.

                          ┌─────────────────┐   
                          │state_getMetadata| 
                          │    RPC method   │
                          └─────────────────┘ 
                                 │   ▲
                                 ▼   │        
                            ┌─────────────┐   
                            │Type registry|  
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
                      
                 (Use bullet points as part above diagram)


Every Substrate chain stores the version number of the metadata system they are using, which makes it useful for front-ends to know how to handle the metadata exposes by a certain block.
As previously mentioned, the latest metadata version (V14) provides a major enhancement to the metadata that a chain is able to generate.
But what if an application wants to interact with blocks that were created with an earlier version than V14?
Well, it would require setting up a front-end interface that follows the older metadata system, whereby custom types would need to be identified and manually included as part of the front-end's code.

Type information bundled in the metadata gives front-ends the ability to dynamically communicate with nodes across different chains, each of which may each expose different calls, events, types and storage.
It also allows libraries to generate almost all of the code needed to communicate with a given Substrate node, giving the possibility for libraries like `subxt` to generate front-end interfaces that are specific to a target chain.

With this system, any runtime can be queried for its available runtime calls, types and parameters.  
The metadata also exposes how a type is expected to be decoded, making it easy for an external application to retrieve and process this information.

## Metadata format

Querying the `state_getMetadata` RPC function will return a vector of SCALE-encoded bytes which is decoded using the [`frame-metadata`](/rustdocs/latest/frame_metadata/index.html) and [`parity-scale-codec`](/rustdocs/latest/parity_scale_codec/index.html) libraries. 

The hex blob returned by the JSON-RPC's `state_getMetadata` method depends on the metadata version, however will generally have the following structure:

- a hard-coded magic number, `0x6d657461`, which represents "meta" in plain text. 
- a 32 bit integer representing the version of the metadata format in use, for example `14` or `0x0e` in hex.
- hex encdoded type and metadata information. 
In V14, this part would contain a registry of type information (generated by the `scale-info` crate). 
In previous versions, this part contained the number of pallets followed by the metadata each pallet exposes.

Here is a condensed version of decoded metadata for a runtime using the V14 metadata system (generated using [`subxt-cli`]()):

```json
[
  1635018093,               // the magic number
  {
    "V14": {                // the metadata version
      "types": {            // type information
              "types": [
              ]
      },
      "pallets": [          // metadata exposes by pallets
      ],
      "extrinsic": {        // the format of an extrinsic  and its signed extensions
        "ty": 111,
        "version": 4,       // the transaction version used to encode and decode an extrinsic
        "signed_extensions": [
        ]
      },
      "ty": 125,            // the type ID for the system pallet
    }
  }
]
```

As described above, the integer `1635018093` is a "magic number" that represents "meta" in plain text. 
The rest of the metadata has two sections: `pallets` and `extrinsic`. 
The `pallets` section contains information about the runtime's pallets, while the `extrinsic` section describes the version of extrinsics that the runtime is using. 
Different extrinsic versions may have different formats, especially when considering [signed extrinsics](/v3/concepts/extrinsics).

### Pallets

Here is a condensed example of a single element in the `pallets` array:

```json
{
  "name": "System",   // name of the pallet, the System pallet for example
  "storage": {        // storage entries
  },
  "calls": [          // index for this pallet's call types
  ],
  "event": [          // index for this pallet's event types
  ],
  "constants": [      // pallet constants
  ],
  "error": [         // index for this pallet's error types
  ],
  "index": 0         // the index of the pallet in the runtime
}
```

Every element contains the name of the pallet that it represents, as well as a `storage` object, `calls` array, `event` array, and `error` array.
If `calls` or `events` are empty, they will be represented as `null` and if `constants` or `errors` are empty, they will be represented as an empty array.

Type indices for each item are just `u32` integers used to access the type information for that item.
For example, the type ID for the `calls` in the System pallet is 145. 
Querying the type ID will give you information about the available calls of the system pallet including the documentation for each call.
For each field, you can access type information and metadata for:

- [Storage metadata](https://docs.substrate.io/rustdocs/latest/frame_metadata/v14/struct.PalletStorageMetadata.html): provides blockchain clients with the information that is required to query [the JSON-RPC's storage function](/rustdocs/latest/sc_rpc/state/struct.StateClient.html#method.storage) to get information for a specific storage item.
- [Call metadata](https://docs.substrate.io/rustdocs/latest/frame_metadata/v14/struct.PalletCallMetadata.html): includes information about the runtime calls are defined by the `#[pallet]` macro including call names, arguments and documentation.
- [Event metadata](https://docs.substrate.io/rustdocs/latest/frame_metadata/v14/struct.PalletEventMetadata.html): provdes the metadata generated by the `#[pallet::event]` macro, including the name, arguments and documentation for a pallet's events
- Constants metadata provides metadata generated by the `#[pallet::constant]` macro, including the name, type and hex encoded value of the constant.
- [Error metadata](https://docs.substrate.io/rustdocs/latest/frame_metadata/v14/struct.PalletErrorMetadata.html): provides metadata generated by the `#[pallet::error]` macro, including the name and documentation for each error type in that pallet. 

### Extrinsic

[Exrinsic metadata](https://docs.substrate.io/rustdocs/latest/frame_metadata/v14/struct.ExtrinsicMetadata.html) is generated by the runtime and provides useful information on how a transaction is formatted.
The returned decoded metadata contains the transaction version and signed extensions, which looks like this:

```json
      "extrinsic": {
        "ty": 111,
        "version": 4,
        "signed_extensions": [
          {
            "identifier": "CheckSpecVersion",
            "ty": 117,
            "additional_signed": 4
          },
          {
            "identifier": "CheckTxVersion",
            "ty": 118,
            "additional_signed": 4
          },
          {
            "identifier": "CheckGenesis",
            "ty": 119,
            "additional_signed": 9
          },
          {
            "identifier": "CheckMortality",
            "ty": 120,
            "additional_signed": 9
          },
          {
            "identifier": "CheckNonce",
            "ty": 122,
            "additional_signed": 34
          },
          {
            "identifier": "CheckWeight",
            "ty": 123,
            "additional_signed": 34
          },
          {
            "identifier": "ChargeTransactionPayment",
            "ty": 124,
            "additional_signed": 34
          }
        ]
      }
```

The type system is composite, which means that each type ID contains a reference to some type or to another type ID that gives access to the associated primitive types.
For example one type we can encode is a `BitVec<Order, Store>` type: to decode it properly we need to know what the `Order` and `Store` types used were, which can be accessed used the "path" in the decoded JSON for that type ID.

## RPC APIs

Substrate comes with the following APIs to interact with a node:

- [`AuthorApi`](https://paritytech.github.io/substrate/master/sc_rpc/author/trait.AuthorApi.html): An API to make calls into a full node, incuding authoring extrinsics and verifying session keys.
- [`ChainApi`](https://paritytech.github.io/substrate/master/sc_rpc/chain/trait.ChainApi.html):	An API to retrieve block header and finality information.
- [`OffchainApi`](https://paritytech.github.io/substrate/master/sc_rpc/offchain/trait.OffchainApi.html): An API for	making RPC calls for off-chain workers.
- [`StateApi`](https://paritytech.github.io/substrate/master/sc_rpc/state/trait.StateApi.html): An API to query information about on-chain state such as runtime version, storage items and proofs.
- [`SystemApi`](https://paritytech.github.io/substrate/master/sc_rpc/system/trait.SystemApi.html): An API to retrieve information about network state, such as connected peers and node roles.	

## Connecting to a node

Querying a Substrate node can either be done by using a Hypertext Transfer Protocol (HTTP) or WebSocket (WS) based JSON-RPC client.
The main advantage of WS (used in most applications) is that a single connection can be reused for many messages to and from a node, whereas a typical HTTP connection allows only for a single message from, and then response to the client at a time.
For this reason, if you want to subscribe to some RPC endpoint that could lead to multiple messages being returned to the client, you must use a websocket connection and not an HTTP one.
Connecting via HTTP is commonly used for fetching data in off-chain workers-learn more about that [here]().

An alternative (and still experimental) way to connect to a Substrate node is by using `Substrate Connect`, which allows applications to spawn their own light clients and connect directly to the exposed JSON-RPC end-point.
These applications would rely on in-browser local memory to establish a connection with the light client. 

## Start building 

Parity maintains the following libraries built on top of the [JSON-RPC API](https://github.com/paritytech/jsonrpc) for interacting with a Substrate node:

- [subxt](./libraries#subxt): provides a way to create an interface for static front-ends built for specific chains. 
- [Polkadot JS API](./libraries#polkadot-js): provides a library to build dynamic interfaces for any Substrate built blockchain.
- [Substrate Connect](./libraries#substrate-connect): provides a library and a browser extension to build applications that connect directly with an in-browser light client created for its target chain. 
As a library that uses the Polkadot JS API, Connect is useful for applications that need to connect to multiple chains, providing end users with a single experience when interacting with multiple chains for the same app.

<!-- TODO: Refactor content below: not sure where it should go. This is more how to use it / how it works.

By default, front-end applications built using any of the existing libraries will connect to an RPC node hosted by some third party.
In rare cases would users of an application be running their own node. 
This implies that developers can never be absolutely certain that their application is displaying correct information about the interactions it has with the node connected through RPC.

Instead of specifying an RPC node, developers just need to include the blockchain's [chain specification](/v3/runtime/chain-specs) and their application will synchronize with the chain.

[Substrate Connect](https://paritytech.github.io/substrate-connect/) is a JavaScript library and browser extension that builds on the [Polkadot JS API](/v3/integration/polkadot-js#polkadot-js-api) to enable developers to build application specific light clients for Substrate chains. 
By using light clients available to all Substrate built blockchains, application developers no longer need to rely on single RPC nodes to allow end-users to interact with their applications. 
This introduces a new paradigm for decentralization: 

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

[ _TODO : Add substrate connect note: https://github.com/substrate-developer-hub/substrate-docs/issues/573_ ]. -->

## Learn more
- Learn how a transaction is formatted in Susbtrate 
- Launch a front-end app using Substrate's front-end template 
- Use a [QR code metadata generation tool](https://github.com/paritytech/metadata-portal) for offline signing devices
- Decode a Substrate node with backwards-compatible metadata with [desub](https://github.com/paritytech/desub)