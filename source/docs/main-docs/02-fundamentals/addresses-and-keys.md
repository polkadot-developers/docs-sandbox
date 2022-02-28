# Addresses and keys

_This article goes over how addresses in Substrate chains are derived: including the SS58 format and different types of network keys._

In the multi-chain future, users should be able to use one public address across different chains.
To enable this, Substrate adopts a novel way to derive multiple accounts under a single public key using the **SS58 address format**.
This provides Substrate built chains with a user-friendly way for users and networks to use their accounts across chains without needing to create new public and private keypairs for each network they interact with.

## SS58 address format

Native to all Substrate chains, the SS58 address format is a simple address format based on Bitcoin's [Base58Check encoding scheme](https://en.bitcoin.it/wiki/Base58Check_encoding).

By using this address formatting technique, networks can also specify custom prefixes for their addresses, making it easy for users to differentiate their derived addresses from another.
In addition, a chain can easily identify, verify and reject invalid accounts without needing to make any storage reads. 

Its key features include:

- It can derive different accounts based on the same public key 256 bit hex.
- Base58 encoding results in 58 alphanumeric string with easily distinguishable symbols, omitting symbols `0`, `O`, `I`, `l` which could lead to confusion.
- The encoded result also contains information on the network information.
- A checksum is used to detect any typographical errors, making it helpful when users make errors in address inputs.

The basic format conforms to the concatenated byte series of the address type, the address and checksum, all passed into a base-58 encoder:

```text
base58encode ( concat ( <address-type>, <address>, <checksum> ) )
```

The `base58encode` function is implemented exactly as defined in Bitcoin and IPFS (see this [wiki](https://en.wikipedia.org/wiki/Base58)), using the same alphabet as both.

## For end users 

From a single public key, you can derive different account types for different registered networks.
The table below shows the different addresses derived from Alice's well known public key.

#### Table: Derived addresses from Alice's public key


| Address type | Address | Prefix |
| ------------ | ------ | ---------- |
| Public key (hex) | 0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d | n/a |
| Genereic Substrate chain (SS58)| 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY | 42 |
| H160 (EVM compatible) | 0xd43593c715Fdd31c61141ABd04a99FD6822c8558 | n/a |
| Polkadot (SS58)| 15oF4uVJwmo4TdGW7VfQxNLavjCXviqxT9S1MgbjMNHr6Sp5 | 0 | 
| Kusama (SS58) | HNZata7iMYWmk5RvZRTiAsSDhV8366zq2YGb3tLH5Upf74F | 2  |

Each address type will carry a consistent feature for the first character of the address.
For instance: all Polkadot addresses will start with 1, Kusama addresses with a capital character and all Substrate chains with 5.

For any other registered chain, you can look up what your account derives to using [this tool](https://polkadot.subscan.io/tools/format_transform).

## For network developers

Network developers can register their networks in a special ss58 registry that becomes part of the `sp_core` create that all Substrate chains depend on. 

Any registered network must specify: 

- A prefix: this must be a unique integer (e.g. 0)
- A network name: this must be some unique string which corresponds to the network using the prefix (e.g. polkadot).
- A display name: a human friendly way to read the name of the network (e.g. Polkadot Relay Chain).
- Token symbol(s): an array of the different symbols corresponding to the network's native tokens. Only chains with multiple instances of the Balances pallet will have multiple tokens reflected here (e.g. ["DOT"]).
- Decimals: an array with the amount of decimals for each native token (e.g. [10]).
- Signing curve: the different signing curves the account uses (e.g. sr25519)

To register your network, follow the steps in [the SS58 repository](https://github.com/paritytech/ss58-registry) (note that you can't register for test networks).
Learn how you can support a custom SS58 prefix without needing to register it in the registry by following [this guide]().

## Generating addresses

The most direct way to verify and generate addresses in Substrate is by using the [Subkey](/v3/tools/subkey) `inspect` subcommand.
This command accepts either the seed phrase for an account, a hex-encoded private key, or an SS58 address as the input URI.
If the input is a valid address, it will return a list containing the corresponding public key (hex), account ID and SS58 values.

Subkey assumes that an address is based on a public/private keypair.
In the case of inspecting an address, it will return the 32 bytes of the account ID.

If you input a valid SS58 value, Subkey will also return a network ID/version value that indicates which network the address has been encoded for.

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

For verifying or creating SS58 addresses in your JavaScript projects, you can utilize the functions provided from the [`ui-keyring` library](https://polkadot.js.org/docs/ui-keyring/start/init) in the PolkadotJS API.

## Account keys

A key pair can represent an account and control funds, like any typical accounts you would expect in other blockchains. 
In Substrate, we can enforce rules over how specific key pairs behave, including specifying custom cryptographic schemes, how dust accounts are managed or specifying special accounts only accessible to certain functions or pallets like on-chain treasuries.

This section outlines some common key types and their features.
In Substrate, account keys are defined generically and made concrete in the runtime.
The keys to these accounts are distinguished by their intended use, not by any underlying cryptographic difference.

If you're looking to learn more about account types in FRAME, [read this article]().

### Nominated Proof of Stake systems

The nominated proof-of-stake (NPoS) algorithm has the most common use for different account keys.
Since node operators (such as validators and nominators) may hold significant amounts of funds, Substrate's [Staking pallet](/v3/runtime/frame#staking) handles some account abstractions that help keep funds as secure as possible.
These are:

- **Stash keys**: a stash account is meant to hold large amounts of funds. 
Its private key should be as secure as possible in a cold wallet.
- **Controller keys**: a controller account signals choices on behalf of the stash account, like payout preferences, but should only hold a minimal amount of funds to pay transaction fees. 
Its private key should be secure as it can affect validator settings, but should be easily available for validator maintenance.
- **Session keys**: these are "hot" keys kept in the validator client and used for signing certain validator operations. 
Session keys should never hold funds.

When creating stash or controller keys, all cryptography supported for normal account keys are also supported.

| Key type | Description | Usage | Generation | 
| -------- | ----------- | ----- | ---------- |
| Stash keys | The stash keys are the public/private key pair that defines a stash account, serving as a savings account for validators. These should be kept offline, in cold storage for increased security. | A stash account should not be used to make frequent transactions. Since it is kept offline, it would designate a controller account to make non-spending decisions or a proxy account to vote in governance on its behalf. | ??? |
| Controller keys | The controller keys define a controller account to signal one's intent to validate or nominate, set preferences like the rewards destination and, in the case of validators, to set their session keys. | A controller account only needs to pay transaction fees, so it only needs a minimal amount of funds and can never be used to spend funds from its stash account. Actions taken by the controller can result in slashing, so it should still be well secured. | ???? |
| Session keys | Session keys are "hot keys" used by validators to sign consensus-related messages and are not meant to be used to control funds. They are also defined generically in the [Session pallet](/rustdocs/latest/pallet_session/index.html) and made concrete in the runtime. To create a Session key, validator operators must attest that a key acts on behalf of their stash account (stake) and nominators. To do so, they create a new session key by signing the key with their controller key. Then, they inform the chain that this key represents their controller key by publishing the Session certificate in a transaction on the chain. | Generate and register the new keys on chain using a [`session.setKeys`](/rustdocs/latest/pallet_session/struct.Module.html#method.set_keys) transaction. Session keys should be changed every session. | `author_rotateKeys`](/rustdocs/latest/sc_rpc/author/trait.AuthorApi.html#tymethod.rotate_keys) RPC call. |

Learn [how sessions keys are implemented](./05-design/implementation-details#session-keys) in Substrate.
