Section: Build
Sub-section: Front-end development
Type: reference 

Front-end development for Substrate runtimes encompasses user-facing interfaces such as browser and desktop applications, as well as client applications for specific hardware requirements and different libraries exist to build these types of interfaces, depending on your needs.
This article aims to explain the process of querying a Substrate node in order to provide background knowledge to help inform these decisions, or to create application specific libraries.

> NOTE: Although it's technically possible to use any protocol to communicate with a Substrate node, this article assumes that the chosen interface is via a node's RPC interface using JSON-RPC.


## Connecting to a node

Querying a Substrate node can be done by using an HTTP or WS based JSONRPC client, however most applications would use the WS protocol to answer JSON-RPC requests.
The main advantage of WS is that a single connection can be reused for many messages to and from a node, whereas a typical HTTP connection allows only for a single message from, and then response to the client at a time.
For this reason, if you want to subscribe to some RPC endpoint that could lead to multiple messages being returned to the client, you must use a websocket connection and not an HTTP one.

An alternative (and still experimental) way to connect to a JSON-RPC  is by using `Smoldot`, which allows applications to spawn their own light clients and connect directly to the exposed JSON-RPC there.

## Metadata system 

Substrate runtimes use a versioned metadata system that helps clients generate the metadata of a node by using the `state_getMetadata` RPC call into the runtime.
This includes information about the storage items, transactions, events, errors and constants that are exposed by that pallet.
The current version (V14) differs significantly from its predecessors as it allows clients to generate information of the types used in the runtime.
This means that if a runtime containts a pallet with some custom type, the type information will be included as part of the metadata returned by `state_getMetadata`.
This rich metadata system is made possible by both [`frame-metadata`](https://docs.substrate.io/rustdocs/latest/frame_metadata/index.html) and [`scale-info`](https://docs.rs/scale-info/latest/scale_info/) crates which all Substrate runtimes using the latest system have.

In order to actually send and receive transactions to and from the node, front-end APIs must implement a SCALE codec library to encode and decode RPC calls.

The general flow of how metadata is generated and exposed looks like:

- Callable pallet functions, as well as types, parameters and documentation are exposed by the runtime.
- The `frame-metadata` crate describes the structure in which the information about how to communicate with the runtime will be provided. 
The information takes the form of a type registry provided by `scale-info`, as well as information about things like which pallets exist (and what the relevant types in the registry are for each pallet).
- The `scale-info` crate is used to annotate types across the runtime, and makes it possible to build a registry of runtime types. This type information is detailed enough that we can use it to find out how to correctly SCALE encode or decode some value for a given type.
- The structure described in `frame-metadata` is populated with information from the runtime, and this is then SCALE encoded and made available via the `state_getMetadata` RPC call.

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

The hex blob returned by the JSON-RPC's `state_getMetadata` method depends on the metadata version, however will generally have the following strcuture:

- a hard-coded magic number, `0x6d657461`, which represents "meta" in plain text. 
- a 32 bit integer representing the version of the metadata format in use, for example `14` or `0x0e` in hex.
- hex encdoded type and metadata information. 
For V14, this part would contain a registry of type information (generated by the `scale-info` crate). 
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
        "version": 4,
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

#### Pallets

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

#### Extrinsic

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

## Libraries 

Different libraries exist for building frontend interfaces for Substrate-based chains and interacting with Substrate runtimes.

- [`subxt`](./libraries#subxt) provides a way to fetch and decode metadata.
- [Polkadot JS API](./libraries#polkadot-js) provides multiple interfaces to interact with a Substrate blockchain.
- [Substrate Connect](./libraries#substrate-connect) provides a library and a browser extension to build applications that connect directly with a target chain. 
As a library that uses the Polkadot JS API, Connect is designed for applications that need to connect to multiple chains, providing end users with a single experience when interacting with multiple chains for the same app.

However, you could use the [JSON-RPC](https://github.com/paritytech/jsonrpc) and [jsonrpsee](https://github.com/paritytech/jsonrpsee) libraries to use a node's RPC interface more directly.
### subxt 

Short for "submit extrinsics", `subxt` is a Rust library that generates an interface to a specific Substrate node based on the current metadata that it returns. 
This allows developers to write user facing client applications with type-safe communication between the node and the code using this generated interface.
As well as submitting extrinsics, `subxt` also allows one to access storage entries, decode blocks and decode the events contained within the blocks.

It's also a good solution for buidling lower level applications, such as non-browser graphic user interfaces or chain-specific CLIs.
The `subxt` library comes with the [`subxt-cli` tool](./reference/command-line-tools) to generate the metadata and runtime API of a target chain, which can also be used as a standalone tool.
### Polkadot JS

The [Polkadot JS project](https://polkadot.js.org/) provides collection of tools, interfaces, and libraries for creating user interfaces for Substrate blockchains in Javascript.
These include:

- The [Polkadot JS API](https://polkadot.js.org/docs/api): a Javascript library for querying and interacting with a Substrate chain.
- The [Polkadot JS extension](https://polkadot.js.org/docs/extension/): an API for interacting with a browser extension.
- The [Polkadot Apps UI](https://github.com/polkadot-js/apps): a basic UI designed to interact with Substrate built chains using the [Polkadot JS API](https://polkadot.js.org/docs/api).

### Substrate connect

Substrate Connect provides a Javascript library for developers to build applications that also act as a light client for their target chain. 
In addition, it provides:
- A browser extension that can be used as an interface between a Substrate chain and a front-end application.
- A way to connect to multiple chains from a single application (web or desktop browser).

By default, front-end applications built using any of the existing libraries will connect to an RPC node hosted by some third party.
In rare cases would users of an application be running their own node. 
This implies that developers can never be absolutely certain that their application is displaying correct information about the interactions it has with the node connected through RPC.

Instead of specifying an RPC node, developers just need to include the blockchain's [chain specification](/v3/runtime/chain-specs) and their application will synchronize with the chain.

<!-- TODO: Refactor content below: not sure where it should go.

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