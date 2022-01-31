Section: Build
Sub-section: Front-end development
Type: reference 

Application development against Substrate runtimes includes user-facing interfaces such as browser applications, desktop applications or applications for low performance hardware environments such as hardware wallets.
Different libraries exist to build these types of applications.
This article aims to describe:

- Types of application providers.
- How extrinsics are sent to and from an application.
- How a Substrate node exposes metadata that can be consumed by an application for any chain.
- The process of querying metadata.
- Existing libraries for writing user facing interfaces. 

## Transport providers

Any client or front-end application requires specifying a provider to answer JSON-RPC requests made to interact with a chain.
This could be for connecting to an existing node such as `wss://rpc.polkadot.io` or connecting to a local port such as `ws://127.0.0.1:9944` when running a chain locally for example. 
Depending on the client implementation, both Websocket and HTTP transport options are available.
Websocket connections are a more secure alternative to HTTP and are better supported among current front-end libraries.
Yet, an alternative way to specify a provider is by using `Smoldot`, which allows applications to spawn their own light clients and connect directly there (still in an experimental phase).

Several [language-specific libraries](./todo-add-link) that enable connections to these transport providers exist already.
However, the APIs for these providers are language-agnostic, making it possible to build front-end APIs in any language.

## Sending and receiving transactions

Specifying an endpoint to connect to will establish a connection to the specified node.
In order to actually send and receive transactions to and from the node, all front-end APIs must implement a SCALE codec library to encode/decode the RPC payloads.

For each pallet, the metadata provides information about the storage items, transactions, events, errors and constants that are exposed by that pallet. 
Substrate automatically generates this metadata for you and makes it available through RPC calls.

[TODO: go more in depth about the basic components that front-end libs need to interact with a node]

## Metadata system 

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

As shown in the above diagram:                          
- Callable pallet functions, as well as types, parameters and documentation are exposed by the runtime.
- The `frame-metadata` API decodes these.
- The `scale-info` crate creates a registry of all types generated from `frame-metadata`. 
- The types in this registry are annotated so they can be encoded and decoded to the SCALE binary format as well as information about how each type will be encoded.

With this system, any runtime can be queried for its available runtime calls, types and parameters.  
The metadata also exposes how a type is expected to be decoded, making it easy for an external application to retrieve and process this information.

[TODO: Update examples from metadata query to V14]

## Querying metadata 

The easiest way to get the metadata is by querying the automatically-generated JSON-RPC function `state_getMetadata`. 
This will return a vector of SCALE-encoded bytes. 
You can decode this using the [`frame-metadata`](/rustdocs/latest/frame_metadata/index.html) and [`parity-scale-codec`](/rustdocs/latest/parity_scale_codec/index.html) libraries.

### Encoded metadata format

The hex blob that is returned by the JSON-RPCs `state_getMetadata` method starts with a hard-coded magic number, `0x6d657461`, which represents "meta" in plain text. 
The next piece of data (`0x0b` in the example above) represents the metadata version; decoding the hexadecimal value `0x0b` yields the
decimal value 11, which is [the version of the Substrate metadata format](/rustdocs/latest/frame_metadata/enum.RuntimeMetadata.html) that the result encodes. 
After the metadata version, the next piece of information encoded in the result field is the number of pallets that inform the blockchain's runtime; in the example above, the hexadecimal value `0x7c` represents the decimal number 31, which is SCALE-encoded by taking its binary representation (`11111` or `0x1F` in hex), shifting it two bits to the left (`1111100`) and encoding that as hex.

The remaining blob encodes [the metadata of each pallet](/rustdocs/latest/frame_metadata/struct.ModuleMetadata.html), which will be reviewed below as well as some [extrinsic metadata](/rustdocs/latest/frame_metadata/struct.ExtrinsicMetadata.html), which is mostly out of the scope of this document.

### Decoded metadata format

Here is a condensed version of decoded metadata:

```json
{
  "magicNumber": 1635018093,
  "metadata": {
    "V12": {
      "modules": [
        {
          // ...
        },
        {
          // ...
        }
      ],
      "extrinsic": {
        "version": 4,
        "signedExtensions": [
          "CheckSpecVersion",
          "CheckTxVersion",
          "CheckGenesis",
          "CheckMortality",
          "CheckNonce",
          "CheckWeight",
          "ChargeTransactionPayment"
        ]
      }
    }
  }
}
```

As described above, the integer `1635018093` is a "magic number" that represents "meta" in plain text. 
The rest of the metadata has two sections: `modules` and `extrinsic`. 
The `modules` section contains information about the runtime's pallets, while the extrinsic section describes the version of extrinsics that the runtime is using. 
Different extrinsic versions may have different formats, especially when considering [signed extrinsics](/v3/concepts/extrinsics).

#### Modules

Here is a condensed example of a single element in the `modules` array:

```json
{
  "name": "System",
  "storage": {
    // ..
  },
  "calls": [
    // ..
  ],
  "events": [
    // ..
  ],
  "constants": [
    // ..
  ],
  "errors": [
    // ..
  ],
  "index": 0
}
```

Every element contains the name of the pallet that it represents, as well as a `storage` object, `calls` array, `events` array, and `errors` array.

<Message
  type={`gray`}
  title={`Note`}
  text={`If \`calls\` or \`events\` are empty, they will be represented as \`null\`; if \`constants\` or
 \`errors\` are empty, they will be represented as an empty array.`}
/>

##### Storage

Here is a condensed example of a single element in the `modules` array that highlights metadata about the module's storage:

```json
{
  "name": "System",
  "storage": {
    "prefix": "System",
    "items": [
      {
        "name": "Account",
        "modifier": "Default",
        "type": {
          "Map": {
            "hasher": "Blake2_128Concat",
            "key": "AccountId",
            "value": "AccountInfo",
            "linked": false
          }
        },
        "fallback": "0x000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
        "documentation": [
          " The full account information for a particular account ID."
        ]
      },
      {
        "name": "ExtrinsicCount"
        // ..
      },
      {
        "name": "AllExtrinsicsLen"
        // ..
      }
    ]
  },
  "calls": [
    /*...*/
  ],
  "events": [
    /*...*/
  ],
  "constants": [
    /*...*/
  ],
  "errors": [
    /*...*/
  ],
  "index": 0
}
```

Every storage item that is defined in a pallet will have a corresponding metadata entry.
Metadata entries like these are generated from [macros](/v3/runtime/macros) using associated types from the [`frame-system`](/rustdocs/latest/frame_system/pallet/trait.Config.html) crate. 
For example:

```rust
#[pallet::config]
pub trait Config: frame_system::Config {
	#[pallet::constant]
	type Foo: Get<u32>;
}
```

[Storage metadata](/rustdocs/latest/frame_metadata/struct.StorageMetadata.html) provides blockchain clients with the information that is required to query [the JSON-RPC's storage function](/rustdocs/latest/sc_rpc/state/struct.StateClient.html#method.storage) to get information for a specific storage item.

##### Dispatchable calls

Metadata for dispatchable calls includes information about the runtime calls are defined by the `#[pallet]` macro. For each call, the metadata includes:

- `name`: Name of the function in the pallet.
- `args`: Arguments in function definition. Includes the name and type of each argument.
- `documentation`: Documentation of the function.

For example:

```rust
#[pallet::call]
impl<T: Config> Pallet<T> {

  /// This function does some thing.
  ///
  /// All documentation details go here.
  pub(super) fn do_something(
    origin: OriginFor<T>,
    #[pallet::compact] thing: T::Something
  ) -> DispatchResultWithPostInfo {
    // ... snip
  }
}
```

This materializes in the metadata as follows:

```json
"calls": [
  {
    "name": "do_something",
    "args": [
      {
        "name": "thing",
        "ty": "Compact<T::Something>"
      }
    ],
    "documentation": [
      " This function does some thing.",
      "",
      " All documentation details go here."
    ]
  }
],
```

##### Events

This metadata snippet is generated from this declaration in `frame-system`:

```rust
#[pallet::event]
#[pallet::metadata(T::AccountId = "AccountId")]
pub enum Event<T: Config> {
  /// An extrinsic completed successfully.
  ExtrinsicSuccess(DispatchInfo, T::AccountId),
  /// An extrinsic failed.
  ExtrinsicFailed(DispatchError, DispatchInfo),
  // ... snip
}

```

Substrate's metadata would describe these events as follows:

```json
"events": [
  {
    "name": "ExtrinsicSuccess",
    "args": [
      "DispatchInfo",
      "AccountId"
    ],
    "documentation": [
      " An extrinsic completed successfully."
    ]
  },
  {
    "name": "ExtrinsicFailed",
    "args": [
      "DispatchError",
      "DispatchInfo"
    ],
    "documentation": [
      " An extrinsic failed."
    ]
  },
],
```

##### Constants

The metadata will include any module constants. 
For example in [`pallet-babe`](https://github.com/paritytech/substrate/blob/master/frame/babe/):

```rust
#[pallet::config]
	#[pallet::disable_frame_system_supertrait_check]
	pub trait Config: pallet_timestamp::Config {
		/// The amount of time, in slots, that each epoch should last.
		/// NOTE: Currently it is not possible to change the epoch duration after
		/// the chain has started. Attempting to do so will brick block production.
		#[pallet::constant]
		type EpochDuration: Get<u64>;
```

The metadata for this constant looks like this:

```json
"constants": [
  {
    "name": "EpochDuration",
    "type": "u64",
    "value": "0x6009000000000000",
    "documentation": [
      " The amount of time, in slots, that each epoch should last.",
      " NOTE: Currently it is not possible to change the epoch duration after",
      "the chain has started. Attempting to do so will brick block production."
    ]
  },
]
```

The metadata also includes constants defined in the runtime's `lib.rs`. 
For example, from Kusama:

```rust
parameter_types! {
    pub const EpochDuration: u64 = EPOCH_DURATION_IN_BLOCKS as u64;
}
```

Where `EPOCH_DURATION_IN_BLOCKS` is a constant defined in `runtime/src/constants.rs`.

##### Errors

Metadata will pull all the possible runtime errors from `#[pallet::error]`. 
For example, from `frame-system`:

```rust
#[pallet::error]
pub enum Error<T> {
        /// The name of specification does not match between the current runtime
        /// and the new runtime.
        InvalidSpecName,
        // ... snip
    }
```

This will expose the following metadata:

```json
"errors": [
  {
    "name": "InvalidSpecName",
    "documentation": [
      " The name of specification does not match between the current runtime",
      " and the new runtime."
    ]
  },
  // ...
]
```

These are errors that could occur during the submission or execution of an extrinsic. 
In this case, the FRAME System pallet is declaring that it may raise the the [`InvalidSpecName` error](/rustdocs/latest/frame_system/pallet/enum.Error.html#variant.InvalidSpecName).

## Libraries 

Different libraries exist for building frontend interfaces for Substrate-based chains and interacting with Substrate runtimes.
- [`subxt`](#subxt) provides a way to fetch the metadata and decode them for you. 
- [Polkadot JS API](#polkadot-js) provides multiple interfaces to interact with a Substrate blockchain.
- [Substrate Connect](#substrate-connect) provides a library to build applications that connect directly with a target chain.

To use the RPC more directly, you can use the [JSONRPC](https://github.com/paritytech/jsonrpc) and [jsonrpsee](https://github.com/paritytech/jsonrpsee) Rust libraries.
### subxt 

`subxt` is a Rust library for submitting extrinsics to a Substrate node.

It provides a way for developers to write user facing client applications for any Substrate chain without the burden of maintaining a list of types. 
It's also a good solution for lower level applications, such as non-browser graphic user interfaces or building an application specific CLI.

As a part of the `subxt` library, the [`subxt-cli` tool](./reference/command-line-tools) was created to generate the metadata and runtime API of a target chain.

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
