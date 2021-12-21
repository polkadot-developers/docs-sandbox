Section: Build
Sub-section: Tools
Type: reference 

Developers building with Substrate can use a number of tools depending on where they are in their development cycle.

### subkey

Subkey is a [public key cryptographic](https://en.wikipedia.org/wiki/Public-key_cryptography) utility that is developed [within Substrate itself](https://github.com/paritytech/substrate/tree/master/bin/utils/subkey).
Its main feature is to generate and inspect key pairs, currently supporting these schemes:

- [sr25519](https://wiki.polkadot.network/docs/en/learn-cryptography): Schorr signatures on
  the Ristretto group
- [ed25519](https://en.wikipedia.org/wiki/EdDSA#Ed25519): SHA-512 (SHA-2) on Curve25519
- [secp256k1](https://en.bitcoin.it/wiki/Secp256k1): ECDSA signatures on secp256k1

All keys in Substrate based networks ([like polkadot](https://wiki.polkadot.network/docs/learn-accounts#address-format))
use the [SS58 address encoding format](/v3/advanced/ss58/) that is the primary user-facing way to interact with keys.

Subkey also allows restoring keys from mnemonics and raw seeds; signing and verifying signatures
on a message; and signing and verifying signatures for encoded transactions.

[Go to documentation](https://github.com/paritytech/substrate/tree/master/bin/utils/subkey#readme).
### sr tool

srtool allows building WASM runtimes in a deterministic way, allowing CIs and users to produce a strictly identical WASM runtime.

[Go to documentation](https://github.com/paritytech/srtool). 

### subxt
A library to submit extrinsics to a Substrate node via RPC.
[Go to documentation](https://github.com/paritytech/substrate-subxt).


### tx wrapper
Tools for FRAME chain builders to publish chain specific offline transaction generation libraries.
[Go to documentation](https://github.com/paritytech/txwrapper-core).

### sub flood
A tool that floods a Substrate node with transactions.
[Go to documentation](https://github.com/paritytech/sub-flood).

### Substrate archive
A tool Run alongside a Substrate-backed chain to index all Blocks, State, and Extrinsic data into PostgreSQL.
[Go to documentation](https://github.com/paritytech/substrate-archive)

### Sidecar
A REST service that makes it easy to interact with blockchain nodes built using Substrate's FRAME framework.
[Go to documentation](https://github.com/paritytech/substrate-api-sidecar).

### Polkadot launch

A simple CLI tool to launch a local Polkadot test network.
[Go to Documentation](https://github.com/paritytech/polkadot-launch).


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
