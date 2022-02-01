---
This article goes over the SS58 format used in Substarte chains and different types of keys used by various participants in Substrate networks.
This is important to for runtime engineers, parachain devops teams and node operators, to understand differnt key types and where to use them.
---

Substrate uses multiple sets of public/private key pairs to represent participants of the network, including validators, nominators and normal users.

As an example, the Substrate node uses a Nominated Proof-of-Stake (NPoS) algorithm to select validators. Validators and nominators may hold significant amounts of funds, so Substrate's [Staking pallet](/v3/runtime/frame#staking) introduces account abstractions that help keep funds as secure as possible.

These abstractions are:

- **Stash Key**: a Stash account is meant to hold large amounts of funds. 
Its private key should be as secure as possible in a cold wallet.
- **Controller Key**: a Controller account signals choices on behalf of the Stash account, like payout preferences, but should only hold a minimal amount of funds to pay transaction fees. 
Its private key should be secure as it can affect validator settings, but will be used somewhat regularly for validator maintenance.
- **Session Keys**: these are "hot" keys kept in the validator client and used for signing certain validator operations. They should never hold funds.

Learn more about validators and nominators in the context of the Substrate's NPoS [Staking pallet](/rustdocs/latest/pallet_staking/index.html).

## Account keys

A key pair can represent an account and control funds, like normal accounts that you would expect from other blockchains. 
In the context of Substrate's [Balances pallet](/rustdocs/latest/pallet_balances/index.html), these accounts
must have a minimum amount (an "existential deposit") to exist in storage.

Account keys are defined generically and made concrete in the runtime.

To continue with our example of Stash and Controller accounts, the keys to these accounts are distinguished by their intended use, not by any underlying cryptographic difference. 
When creating Stash or Controller keys, all cryptography supported for normal account keys are also supported.

### Stash keys

The Stash keys are the public/private key pair that defines a Stash account. 
This account is like a "savings account" in that you should not make frequent transactions from it. Therefore, its private key should be treated with the utmost security, for example protected in a safe or layers of hardware security.

Since the Stash key is kept offline, it designates a Controller account to make non-spending decisions with the weight of the Stash account's funds. 
It can also designate a Proxy account to vote in governance on its behalf.

### Controller keys

The Controller keys are the public/private key pair that defines a Controller account. 
In the context of Substrate's NPoS model, the Controller key will signal one's intent to validate or
nominate.

The Controller key is used to set preferences like the rewards destination and, in the case of validators, to set their Session keys. 
The Controller account only needs to pay transaction fees, so it only needs a minimal amount of funds.

The Controller key can never be used to spend funds from its Stash account. 
However, actions taken by the Controller can result in slashing, so it should still be well secured.

<!-- Todo: make list of known key types: https://github.com/paritytech/substrate/blob/9d1790636e55a3456bdab91ff2d0e059878d3c42/primitives/core/src/crypto.rs#L1102-L1126. -->

## Session keys 

Substrate provides the [Session pallet](/rustdocs/latest/pallet_session/index.html),
which allows validators to manage their session keys.

Session keys are "hot keys" that are used by validators to sign consensus-related messages. They are
not meant to be used as account keys that control funds and should only be used for their intended
purpose. They can be changed regularly; your Controller only needs to create new session keys by
signing a session's public key and broadcasting this certificate via an extrinsic. Session keys are also
defined generically and made concrete in the runtime.

To create a Session key, validator operators must attest that a key acts on behalf of their Stash
account (stake) and nominators. To do so, they create a new session keys by signing the key with their
Controller key. Then, they inform the chain that this key represents their Controller key by
publishing the Session certificate in a transaction on the chain.

## Implementation

Session keys are used by validators to sign consensus-related messages. [`SessionKeys`](/rustdocs/latest/sp_session/trait.SessionKeys.html)
is a generic, indexable type that is made concrete in the runtime.

You can declare any number of Session keys. For example, the default Substrate node uses four. Other
chains could have more or fewer depending on what operations the chain expects its validators to
perform.

In practice, validators amalgamate all of the session public keys into a single object, sign the set
of public keys with a "Controller" account, and submit a transaction to register the keys on chain.
This on-chain registration links a validator _node_ with an _account_ that holds funds. As such,
that account can be credited with rewards or slashed based on the node's behavior.

The runtime declares what session keys will be implemented with the help of a macro. An
[example](/rustdocs/latest/src/node_runtime/lib.rs.html#435-442):

```rust
impl_opaque_keys! {
    pub struct SessionKeys {
        pub grandpa: Grandpa,
        pub babe: Babe,
        pub im_online: ImOnline,
        pub authority_discovery: AuthorityDiscovery,
    }
}
```

The actual cryptographic curve that each key uses gets defined in `primitives`. For example,
[BABE's key uses sr25519](/rustdocs/latest/src/sp_consensus_babe/lib.rs.html#44-47):

```rust
mod app {
	  use sp_application_crypto::{app_crypto, key_types::BABE, sr25519};
	  app_crypto!(sr25519, BABE);
}
```

Typically, these keys are also initially configured in the genesis state to launch your
chain with pre-established validators. You can see this demonstrated in the
[private network tutorial](/tutorials/v3/private-network/).

### Strongly typed wrappers

The Session keys from a Substrate node could use the same cryptography, but serve _very_ different purposes
in your runtime logic. To prevent the wrong key being used for the wrong operation, strong
Rust types wrap these keys, keeping them incompatible with one another and ensuring they are only
used for their intended purpose.

## Generation and use

As a node operator, you can generate keys using the RPC call
[`author_rotateKeys`](/rustdocs/latest/sc_rpc/author/trait.AuthorApi.html#tymethod.rotate_keys).
You will then need to register the new keys on chain using a [`session.setKeys`](/rustdocs/latest/pallet_session/struct.Module.html#method.set_keys) transaction.

<Message
  type={`yellow`}
  title={`Important`}
  text={`For increased security, session keys should be changed every session. This can be done by creating 
  new session keys, putting the session public keys into an extrinsic, and applying this extrinsic on chain.
  `}
/>

Since session keys are hot keys that must be kept online, the individual keys should **not** be used to
control funds. All the logic for handling session keys is in the Substrate client, primitives, and
Session pallet. If one of the Session keys is compromised, the attacker could commit slashable
behavior.

## SS58 format 

SS58 is a simple address format designed for Substrate based chains.
There's no problem with using other address formats for a chain, but this serves as a robust default.
It is heavily based on Bitcoin's Base-58-check format with a few alterations.

The basic idea is a base-58 encoded value which can identify a specific account on the Substrate chain.
Different chains have different means of identifying accounts.
SS58 is designed to be extensible for this reason.

## Format in detail

This page outlines the implementation of [`Ss58Codec` in Substrate](https://paritytech.github.io/substrate/master/sp_core/crypto/trait.Ss58Codec.html).
Also of note is the canonical [SS58 registry](https://github.com/paritytech/ss58-registry) of address type mappings to various networks described below.

### Basic format

The basic format conforms to:

```text
base58encode ( concat ( <address-type>, <address>, <checksum> ) )
```

That is, the concatenated byte series of address type, address and checksum then passed into a base-58 encoder.
The `base58encode` function is implemented exactly as [defined](https://en.wikipedia.org/wiki/Base58) in Bitcoin and IPFS, using the same alphabet as both.

### Address type

The `<address-type>` is one or more bytes that describe the precise format of the following bytes.

Currently, there exist several valid values:

- 00000000b..=00111111b (0..=63 inclusive): Simple account/address/network identifier.
  The byte can be interpreted directly as such an identifier.
- 01000000b..=01111111b (64..=127 inclusive): Full address/address/network identifier.
  The lower 6 bits of this byte should be treated as the upper 6 bits of a 14 bit identifier value, with the lower 8 bits defined by the following byte.
  This works for all identifiers up to 2\*\*14 (16,383).
- 10000000b..=11111111b (128..=255 inclusive): Reserved for future address format extensions.

The latter (42) is a "wildcard" address that is meant to be equally valid on all Substrate networks that support fixed-length addresses.
For production networks, however, a network-specific version may be desirable to help avoid the key-reuse between networks and some of the problems that it can cause.
Substrate nodes will default to printing keys in address type 42, though alternative Substrate-based node implementations (e.g. Polkadot) may elect to default to some other type.

### Address Formats for Substrate

There are 16 different address formats, identified by the length (**in bytes**) of the total payload (i.e. including the checksum).

| Total | Type | Raw Account | Checksum |
| :---: | :--: | :---------: | :------: |
|   3   |  1   |      1      |    1     |
|   4   |  1   |      2      |    1     |
|   5   |  1   |      2      |    2     |
|   6   |  1   |      4      |    1     |
|   7   |  1   |      4      |    2     |
|   8   |  1   |      4      |    3     |
|   9   |  1   |      4      |    4     |
|  10   |  1   |      8      |    1     |
|  11   |  1   |      8      |    2     |
|  12   |  1   |      8      |    3     |
|  13   |  1   |      8      |    4     |
|  14   |  1   |      8      |    5     |
|  15   |  1   |      8      |    6     |
|  16   |  1   |      8      |    7     |
|  17   |  1   |      8      |    8     |
|  34   |  1   |     32      |    2     |

## Checksum types

Several potential checksum strategies exist within Substrate, giving different length and longevity guarantees.
There are two types of checksum preimages (known as SS58 and AccountID) and many different checksum lengths (1 to 8 bytes).

In all cases for Substrate, the [Blake2-256](<https://en.wikipedia.org/wiki/BLAKE_(hash_function)>) hash function is used.
The variants simply select the preimage used as the input to the hash function and the number of bytes taken from its output.

The bytes used are always the left most bytes.
The input to be used is the non-checksum portion of the SS58 byte series used as input to the base-58 function, i.e. `concat( <address-type>, <address> )`.
A context prefix of `0x53533538505245`, (the string `SS58PRE`) is prepended to the input to give the final hashing preimage.

The advantage of using more checksum bytes is simply that more bytes provide a greater degree of protection against input errors and index alteration at the cost of widening the textual address by an extra few characters.
For the account ID format, this is insignificant and therefore no 1-byte alternative is provided.
For the shorter account-index formats, the extra byte represents a far greater portion of the final address, so it is left for further up the stack (though not necessarily the user themselves) to determine the best tradeoff for their purposes.

## Simple/full address types and account/address/network identifiers

[The registry](https://github.com/paritytech/ss58-registry) expresses the status of the account/address/network **identifiers**.

**Identifiers** up to value 64 may be expressed in a **simple** format address, in which the least significant byte (LSB) of the identifier value is expressed as the first byte of the encoded address.

For identifiers of between 64 and 16,383, the **full** format address must be used.

This encoding is slightly fiddly since we encode as little endian (LE), yet the first two bits (which should encode 64s and 128s) are already used up with the necessary `01` prefix.
We treat the first two bytes as a 16 bit sequence, and we disregard the first two bits of that (since they're already fixed to be `01`).
With the remaining 14 bits, we encode our identifier value as LE, with the assumption that the two missing higher order bits are zero.
This effectively spreads the low-order byte across the boundary between the two bytes.

Thus the 14-bit identifier `0b00HHHHHH_MMLLLLLL` is expressed in the two bytes as:

- `0b01LLLLLL`
- `0bHHHHHHMM`

Identifiers of 16384 and beyond are not currently supported.

## Validating addresses

There are several ways to verify that a value is a valid SS58 address.

### Subkey

You can use the [Subkey](/v3/tools/subkey) `inspect` subcommand, which accepts the seed phrase, a hex-encoded private key, or an SS58 address as the input URI.
If the input is a valid address, it will return a list containing the corresponding public key (hex), account ID and SS58 values.

Subkey assumes that an address is based on a public/private keypair.
In the case of inspecting an address, it will return the 32 byte account ID.
Not all addresses in Substrate-based networks are based on keys.

<Message
  type={`gray`}
  title={`Note`}
  text={`If you input a valid SS58 value, Subkey will also return a network ID/version value
that indicates for which network the address has been encoded.`}
/>

```bash
# A valid address.
$ subkey inspect "12bzRJfh7arnnfPPUZHeJUaE62QLEwhK48QnH9LXeK2m1iZU"
Public Key URI `12bzRJfh7arnnfPPUZHeJUaE62QLEwhK48QnH9LXeK2m1iZU` is account:
  Network ID/version: polkadot
  Public key (hex):   0x46ebddef8cd9bb167dc30878d7113b7e168e6f0646beffd77d69d39bad76b47a
  Account ID:         0x46ebddef8cd9bb167dc30878d7113b7e168e6f0646beffd77d69d39bad76b47a
  SS58 Address:       12bzRJfh7arnnfPPUZHeJUaE62QLEwhK48QnH9LXeK2m1iZU

# An invalid address.
$ subkey inspect "12bzRJfh7arnnfPPUZHeJUaE62QLEwhK48QnH9LXeK2m1iZUInvalidAddress"
Invalid phrase/URI given
```

### PolkadotJS API

For verifying an address in your JavaScript projects, you can utilize the functions built into the [PolkadotJS API](https://polkadot.js.org/docs/api/).

```javascript
// Import Polkadot.js API dependencies.
const { decodeAddress, encodeAddress } = require('@polkadot/keyring')
const { hexToU8a, isHex } = require('@polkadot/util')

// Specify an address to test.
const address = '<addressToTest>'

// Check address.
const isValidSubstrateAddress = () => {
  try {
    encodeAddress(isHex(address) ? hexToU8a(address) : decodeAddress(address))

    return true
  } catch (error) {
    return false
  }
}

// Query result.
const isValid = isValidSubstrateAddress()
console.log(isValid)
```

## Learn more

- [`Ss58Codec` in Substrate](https://paritytech.github.io/substrate/master/sp_core/crypto/trait.Ss58Codec.html)
- The canonical [SS58 registry](https://github.com/paritytech/ss58-registry)
- [Polkadot-js API on GitHub](https://github.com/polkadot-js/api)
- [Install Subkey](/v3/tools/subkey#installation)
- [Ecosystem client
  libs](https://substrate.io/ecosystem/resources/awesome-substrate/#client-libraries) and [tools(https://substrate.io/ecosystem/resources/awesome-substrate/#tools)]

- Follow our
  [tutorial to create a local network and generate keys](/tutorials/v3/private-network).
- Visit the reference docs for the
  [session keys runtime API](/rustdocs/latest/sp_session/trait.SessionKeys.html).
- Take a look at the default
  [session keys in the Substrate node runtime](/rustdocs/latest/node_runtime/struct.SessionKeys.html).
- Take a look at the
  [`sp_application_crypto`](/rustdocs/latest/sp_application_crypto/index.html),
  crate, used for constructing application-specific strongly typed crypto wrappers.

## Learn more

- Learn more about Session Keys in the [this article](/v3/concepts/session-keys).
- Learn about the [cryptography used within Substrate](/v3/advanced/cryptography).
- Follow our
  [tutorial to create a local network and generate keys](/tutorials/v3/private-network).
- Visit the reference docs for the
  [session keys runtime API](/rustdocs/latest/sp_session/trait.SessionKeys.html).
- Take a look at the default
  [session keys in the Substrate node runtime](/rustdocs/latest/node_runtime/struct.SessionKeys.html).
