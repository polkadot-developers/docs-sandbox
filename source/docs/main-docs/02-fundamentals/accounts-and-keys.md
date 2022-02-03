_This article goes over the SS58 format used in Substrate chains and different types of keys used by various participants in Substrate networks. This is important for runtime engineers, parachain devops teams and node operators, to understand different key types, how to generate them and where to use them._


Using the SS58 account ID format, Substrate uses multiple sets of public/private key pairs to represent participants of the network, including validators, nominators and normal users.
See the list of known key types [here](https://github.com/paritytech/substrate/blob/9d1790636e55a3456bdab91ff2d0e059878d3c42/primitives/core/src/crypto.rs#L1102-L1126).

## SS58 format 

SS58 is a simple address format which serves as a robust default designed for Substrate based chains.
It is heavily based on Bitcoin's Base-58-check format with a few alterations.

The basic idea is a base-58 encoded value which can identify a specific account on the Substrate chain.
Different chains have different means of identifying accounts.
SS58 is designed to be extensible for this reason.

The basic format conforms to:

```text
base58encode ( concat ( <address-type>, <address>, <checksum> ) )
```

That is, the concatenated byte series of address type, address and checksum then passed into a base-58 encoder.
The `base58encode` function is implemented exactly as [defined in Bitcoin and IPFS](https://en.wikipedia.org/wiki/Base58), using the same alphabet as both.

### Address formats in Substrate

There are 16 different address formats, identified by the length (**in bytes**) of the total payload, including the checksum.

| Address type (`<address-type>`) | Raw Account (`<address>`) | Checksum (`<checksum>`) | Total bytes |
| :---: | :--: | :---------: | :------: |
|  1   |      1      |    1     |  3   |
|  1   |      2      |    1     |  4   |
|  1   |      2      |    2     |  5   | 
|  1   |      4      |    1     |  6   |
|  1   |      4      |    2     |  7   |
|  1   |      4      |    3     |  8   |
|  1   |      4      |    4     |  9   |
|  1   |      8      |    1     |  10   | 
|  1   |      8      |    2     |  11   | 
|  1   |      8      |    3     |  12   | 
|  1   |      8      |    4     |  13   |
|  1   |      8      |    5     |  14   | 
|  1   |      8      |    6     |  15   |  
|  1   |      8      |    7     |  16   | 
|  1   |      8      |    8     |  17   |  
|  1   |     32      |    2     |  35   |


Read more about the implementation details of `Ss58Codec` in Substrate [here](./05-design/implementation-details#ss58).
### Validating addresses

The most direct way to verify and generate addresses in Substrate is by using the [Subkey](/v3/tools/subkey) `inspect` subcommand, 
This command accepts either the seed phrase for an account, a hex-encoded private key, or an SS58 address as the input URI.
If the input is a valid address, it will return a list containing the corresponding public key (hex), account ID and SS58 values.

Subkey assumes that an address is based on a public/private keypair.
In the case of inspecting an address, it will return the 32 byte account ID.
Not all addresses in Substrate-based networks are based on keys.

If you input a valid SS58 value, Subkey will also return a network ID/version value that indicates for which network the address has been encoded.

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

## Acount keys

The most common use of different account keys is found in the nominated proof-of-stake (NPoS) algorithm to select validators. 
Validators and nominators may hold significant amounts of funds, so Substrate's [Staking pallet](/v3/runtime/frame#staking) handles some account abstractions that help keep funds as secure as possible.

These abstractions are:

- **Stash keys**: a stash account is meant to hold large amounts of funds. 
Its private key should be as secure as possible in a cold wallet.
- **Controller keys**: a controller account signals choices on behalf of the stash account, like payout preferences, but should only hold a minimal amount of funds to pay transaction fees. 
Its private key should be secure as it can affect validator settings, but will be used somewhat regularly for validator maintenance.
- **Session keys**: these are "hot" keys kept in the validator client and used for signing certain validator operations. These should never hold funds.

A key pair can represent an account and control funds, like normal accounts that you would expect from other blockchains. 
In the context of Substrate's [Balances pallet](/rustdocs/latest/pallet_balances/index.html), these accounts must have a minimum amount (an "existential deposit") to exist in storage.

In Substrate, account keys are defined generically and made concrete in the runtime.
The keys to these accounts are distinguished by their intended use, not by any underlying cryptographic difference. 
When creating stash or controller keys, all cryptography supported for normal account keys are also supported.

### Stash keys

The stash keys are the public/private key pair that defines a stash account. 
This account is like a "savings account" in that you should not make frequent transactions from it. 
Therefore, its private key should be treated with the utmost security, for example protected in a safe or layers of hardware security.

Since the stash key is kept offline, it designates a controller account to make non-spending decisions with the weight of the stash account's funds. 
It can also designate a Proxy account to vote in governance on its behalf.

### Controller keys

The controller keys are the public/private key pair that defines a controller account. 
In the context of Substrate's NPoS model, the controller key will signal one's intent to validate or nominate.

The controller key is used to set preferences like the rewards destination and, in the case of validators, to set their Session keys. 
The controller account only needs to pay transaction fees, so it only needs a minimal amount of funds.

The controller key can never be used to spend funds from its stash account. 
However, actions taken by the controller can result in slashing, so it should still be well secured.

### Session keys 

Substrate provides the [Session pallet](/rustdocs/latest/pallet_session/index.html) for validators to manage their session keys.

Session keys are "hot keys" that are used by validators to sign consensus-related messages. 
They are not meant to be used to control funds and should only be used for signing messages. 
They can be changed regularly; your controller only needs to create new session keys by signing a session's public key and broadcasting this certificate via an extrinsic. 
Session keys are also defined generically and made concrete in the runtime.

To create a Session key, validator operators must attest that a key acts on behalf of their stash account (stake) and nominators. 
To do so, they create a new session key by signing the key with their controller key. 
Then, they inform the chain that this key represents their controller key by publishing the Session certificate in a transaction on the chain.

#### Generation and use

As a node operator, you can generate session keys using the RPC call [`author_rotateKeys`](/rustdocs/latest/sc_rpc/author/trait.AuthorApi.html#tymethod.rotate_keys).
You will then need to register the new keys on chain using a [`session.setKeys`](/rustdocs/latest/pallet_session/struct.Module.html#method.set_keys) transaction.

For increased security, session keys should be changed every session. 
This can be done by creating new session keys, putting the session's public keys into an extrinsic, and applying this extrinsic on chain.

Since session keys are hot keys that must be kept online, the individual keys should **not** be used to control funds. 
All the logic for handling session keys is in the Substrate client, primitives, and Session pallet. 
If one of the Session keys is compromised, the attacker could commit slashable behavior.

Learn [how sessions keys are implemented](./05-design/implementation-details#session-keys) in Substrate.

## Learn more

- The canonical [SS58 registry](https://github.com/paritytech/ss58-registry)
- [Ecosystem client libs](https://substrate.io/ecosystem/resources/awesome-substrate/#client-libraries) and [tools](https://substrate.io/ecosystem/resources/awesome-substrate/#tools)
- Follow our [tutorial to create a local network and generate keys](/tutorials/v3/private-network).
- Visit the reference docs for the [session keys runtime API](/rustdocs/latest/sp_session/trait.SessionKeys.html).
- Take a look at the default [session keys in the Substrate node runtime](/rustdocs/latest/node_runtime/struct.SessionKeys.html).
- Take a look at the [`sp_application_crypto`](/rustdocs/latest/sp_application_crypto/index.html), crate, used for constructing application-specific strongly typed crypto wrappers.
- Learn about the [cryptography used within Substrate](/v3/advanced/cryptography).
