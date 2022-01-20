---
title: Metadata
slug: /v3/runtime/metadata
version: '3.0'
section: docs
category: runtime
keywords:
---

Blockchains you build on Substrate expose metadata in order to make it easier to interact with them. This metadata is separated by the different [pallets](/v3/runtime/frame#pallets) that inform your blockchain.

Since the runtime of a Substrate blockchain is an evolving part of the blockchain's state,
blockchain metadata is stored on a per-block basis. Be aware that querying the metadata for an older block (with an archive node, for example) could result in acquiring out-of-date metadata that is not compatible with a blockchain's current state. 

As described in the [Upgrades documentation](/v3/runtime/upgrades), developers building on top of Substrate chains can expect that the metadata for a chain should only change when the chain's
[runtime `spec_version`](/rustdocs/latest/sp_version/struct.RuntimeVersion.html#structfield.spec_version) changes.

All examples in this document were taken from block 1,768,321 on Kusama. 
You can look at the [full metadata](https://gist.github.com/insipx/db5e49c0160b1f1bd421a3c34fefdf48) before reading the rest of this document and continue to refer to it as you proceed.

## Get the metadata

There are a number of language-specific libraries that you can use to fetch metadata from a
Substrate node, as well as language-agnostic HTTP and WebSocket APIs.

### Rust

Once decoded, the structure may be serialized into JSON using [`serde`](https://serde.rs/). 


### JavaScript

You can use the following code snippets to fetch the metadata in this
[Polkadot-JS App - JavaScript page](https://polkadot.js.org/apps/#/js):

```javascript
const metadata = await api.rpc.state.getMetadata()
console.log('version: ' + metadata.version)
console.log('Magic number: ' + metadata.magicNumber)
console.log('Metadata: ' + JSON.stringify(metadata.asLatest.toHuman(), null, 2))
```

### HTTP & websocket APIs

Substrate nodes expose [a JSON-RPC API](/rustdocs/latest/sc_rpc/index.html) that you can access by way of **HTTP** or **WebSocket** requests. 
The message to [request metadata](/rustdocs/latest/sc_rpc/state/struct.StateClient.html#method.metadata) from a node looks like this:

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "state_getMetadata",
  "params": []
}
```

You can leave `params` empty, or if you want to fetch the metadata for a specific block, provide the block's hash:

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "state_getMetadata",
  "params": [
    "0xca15c2f1e1540517697b6b5f2cc6bc0c60876a1a1af604269b7215970798bbed"
  ]
}
```

In the example above, `0xca15c2f1e1540517697b6b5f2cc6bc0c60876a1a1af604269b7215970798bbed` is the
hash of block 1,768,321.

The response has the following format:

```json
{
  "jsonrpc": "2.0",
  "result": "0x6d6574610b7c1853797374656d011853797374656d3c1c4163636f756e7401010230543a3a4163636f756e744964944163...",
  "id": 1
}
```

The `result` field contains the blockchain metadata as a [SCALE-encoded](/v3/advanced/scale-codec)
hexadecimal string. The example above represents the actual value that is returned for block
1,768,321; you can check for yourself by using a WebSocket client to query a node. Continue reading
to learn more about the format of this encoded blob as well as
[its decoded format](https://gist.githubusercontent.com/insipx/db5e49c0160b1f1bd421a3c34fefdf48/raw/2c33ff080bec84f0627610124c732deb30a0adc7/meta_block_1768321.json).

## Metadata formats

This section will briefly review the SCALE-encoded metadata that is represented as a hexadecimal
string before taking a more detailed look at the metadata's decoded format.


## Next steps

### Learn more

- [Storage](/v3/runtime/storage)
- [SCALE](/v3/advanced/scale-codec)
- [Macros](/v3/runtime/macros)
- [Events](/v3/runtime/events-and-errors)
- [Extrinsics](/v3/concepts/extrinsics)

### References

- [Metadata](/rustdocs/latest/frame_metadata/index.html)
- [FRAME v2 macro documentation](/rustdocs/latest/frame_support/attr.pallet.html)
