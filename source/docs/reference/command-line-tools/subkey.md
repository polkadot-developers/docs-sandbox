# subkey

The `subkey` program is a key generation and management utility that is included in the Substrate repository.
You can use the `subkey` program to perform the following tasks:

* Generate and inspect cryptographically-secure public and private key pairs.
* Restore keys from mnemonics and raw seeds.
* Sign and verify signatures on a message.
* Sign and verify signatures for encoded transactions.

## Signature schemes

The `subkey` program currently supporting the following signature schemes:

- [sr25519](https://wiki.polkadot.network/docs/en/learn-cryptography): Schorr signatures on the Ristretto group.
- [ed25519](https://en.wikipedia.org/wiki/EdDSA#Ed25519): SHA-512 (SHA-2) on Curve25519.
- [secp256k1](https://en.bitcoin.it/wiki/Secp256k1): ECDSA signatures on secp256k1.

In Substrate-based networks, the `sr25519` encoded keys are used to produce
[SS58 addresses]() as the public keys for interacting with the blockchain.

## Installation

You can download, install, an compile `subkey` using `cargo` without cloning the full Substrate repository.
However, you must add Substrate build dependencies to your environment before you can install `subkey` as a standalone binary.
To ensure dependencies are available, you can build the `subkey` binary from a clone of the Substrate repository.

To install and compile the `subkey` program:

1. Open a terminal shell, if necessary.

1. Verify that you have the Rust compiler and toolchain, if necessary.

1. Clone the Substrate repository, if necessary, by running the following command:

    ```bash
    git clone https://github.com/paritytech/substrate.git
    ```

1. Change to the root directory of the Substrate repository by running the following command:
    
    ```bash
    cd substrate
    ```

1. Compile the `subkey` program using the `nightly` toolchain by running the following command:
    
    ```bash
    cargo +nightly build --package subkey --release
    ```

    Because of the number of packages involved, compiling the node can take several minutes.

1. Verify that your node is ready to use and information about the options available by running the following command:
    
    ```bash
    ./target/release/subkey --help
    ```

## Basic command usage

The basic syntax for running `subkey` commands is:

```
subkey [subcommand] [flag]
```

Depending on the subcommand you specify, additional arguments, options, and flags might apply or be required. 
To view usage information for a specific `subkey` subcommand, specify the subcommand and the `--help` flag. 
For example, to see usage information for `subkey inspect`, you can run the following command:

```
subkey inspect --help
```

### Flags

You can use the following optional flags with the `subkey` command.

| Flag   | Description
| ------ | -----------
| -h, --help | Displays usage information.
| -V, --version | Displays version information.

### Subcommands

You can use the following subcommands with the `subkey` command. 
For reference information and examples that illustrate using subkey subcommands, select an appropriate command.

| Command | Description
| ------- | -----------
| [`generate`]() | Generates a random account key.
| [`generate-node-key`]() | Generates a random node `libp2p` secret key. You can save the secret key to a file or display it as standard output (`stdout`).
| [`help`]() | Displays usage message for `subkey` or for a specified subcommand.
| [`inspect`]() | Displays the public key and SS58 address for the secret URI you specify.
| [`inspect-node-key`]() | Displays the peer ID that corresponds with the secret node key in the file name you specify.
| [`sign`]() | Signs a message with the secret key you specify.
| [`vanity`]() | Generates a seed that provides a vanity address.
| [`verify`]() | Verifies the signature for a message is valid for the public or secret key you specify.

### Output

Depending on the subcommand you specify, the output from the subkey program displays some or all of the following information:

| This field | Contains
| ---------- | ----------
| Secret phrase | A series of English words that encodes the secret key in a human-friendly way. This series of words—also referred to as a mnemonic phrase or seed phrase—can be used to recover a secret key if the correct set of words are provided in the correct order.
| Secret Seed | The minimum information necessary to restore a key pair. The secret seed is also sometimes referred to as a private key or raw seed. All other information is calculated from this value.
| Public Key (hex) | The public half of the cryptographic key pair in hexadecimal format.
| Public Key (SS58) | The public half of the cryptographic key pair in SS58 encoding.
| Account ID | An alias for the public key in hexadecimal format.
| SS58 Address | An SS58-encoded public address based on the public key.

### Examples

To display version information for the `subkey` program, run the following command:

```
subkey --version
```

## subkey generate

Use the `subkey generate` command to generate public and private keys and account addresses.
You can use command-line options to generate keys with different signature schemes or mnemonic phrases with more or fewer words.

#### Basic usage

```
subkey generate [FLAGS] [OPTIONS]
```

#### Flags

You can use the following optional flags with the `subkey generate` command.

| Flag   | Description
| ------ | -----------
| -h, --help | Displays usage information.
| --password-interactive | Enables you to enter the password for accessing the keystore interactively in the terminal.
| -V, --version | Displays version information.

#### Options

You can use the following command-line options with the `subkey generate` command.

| Option   | Description
| -------- | -----------
| --keystore-path <path> | Specifies a custom keystore path.
|--keystore-uri <keystore-uri> | Specifies a custom URI to connect to for keystore-services
| -n, --network <network> | Specifies the network address format to use. For example, `kusama` or `polkadot`. For a complete list of networks supported, see the online usage information.
| --output-type <format> | Specifies the output format to use. Valid values are Json and Test. The default output format is Text
| --password <password> | Specifies the password used by the keystore. This option enables you to append an extra secret to the seed.
| --password-filename <path> | Specifies the name of a file that contains the password used by the keystore.
| --scheme <SCHEME>                cryptography scheme [default: Sr25519]  [possible values: Ed25519, Sr25519,
                                         Ecdsa]
    -w, --words <WORDS>                  The number of words in the phrase to generate. One of 12 (default), 15, 18, 21
                                         and 24


Generate an sr25519 key by running:

```bash
subkey generate
```

```text
Secret phrase `caution juice atom organ advance problem want pledge someone senior holiday very` is account:
  Secret seed:       0xc8fa03532fb22ee1f7f6908b9c02b4e72483f0dbd66e4cd456b8f34c6230b849
  Public key (hex):  0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  Public key (SS58): 5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
  Account ID:        0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  SS58 Address:      5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
```


## subkey inspect

Use the `subkey inspect` command to recalculate the public key and public address for specified secret key or mnemonic phrase.
For example, to inspect the public keys derived from a mnemonic phrase, you can run a command similar to the following:

```bash
subkey inspect "caution juice atom organ advance problem want pledge someone senior holiday very"
```

The command displays output similar to the following:

```text
Secret phrase `caution juice atom organ advance problem want pledge someone senior holiday very` is account:
  Secret seed:       0xc8fa03532fb22ee1f7f6908b9c02b4e72483f0dbd66e4cd456b8f34c6230b849
  Public key (hex):  0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  Public key (SS58): 5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
  Account ID:        0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  SS58 Address:      5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
```

You can also use the secret seed directly by running a command similar to the following:

```bash
subkey inspect 0xc8fa03532fb22ee1f7f6908b9c02b4e72483f0dbd66e4cd456b8f34c6230b849
```


We will use this seed as the [example](#example-seed) for the rest of this document.

---

For more security, use `--words 24` (supports 12, 15, 18, 21, and 24):

_Command:_

```bash
subkey generate --words 24
```

_Output:_

```text
Secret phrase `voice become refuse remove ordinary recall humble purity shock fetch open scale knee above axis blossom differ bamboo ski drip forest fade ill door` is account:
  Secret seed:       0xed26c8119b4872da9e5c2d4b679c82a7fbbabed22cb9f61d5fd4d7806ee5b014
  Public key (hex):  0x182a8385912e206910a11dfcfbadcc018d73ff3febe36f9eb85774e77591a314
  Public key (SS58): 5CcPdpUAhRvGjyxcpnL7emi7SruQ98eP7mC8cDNkjCKfXNZ7
  Account ID:        0x182a8385912e206910a11dfcfbadcc018d73ff3febe36f9eb85774e77591a314
  SS58 Address:      5CcPdpUAhRvGjyxcpnL7emi7SruQ98eP7mC8cDNkjCKfXNZ7
```

Use the `--scheme` option to generate an ed25519 key:

_Command:_

```bash
subkey generate --scheme ed25519
```

_Output:_

```text
Secret phrase `voice become refuse remove ordinary recall humble purity shock fetch open scale knee above axis blossom differ bamboo ski drip forest fade ill door` is account:
  Secret seed:       0xed26c8119b4872da9e5c2d4b679c82a7fbbabed22cb9f61d5fd4d7806ee5b014
  Public key (hex):  0x182a8385912e206910a11dfcfbadcc018d73ff3febe36f9eb85774e77591a314
  Public key (SS58): 5CcPdpUAhRvGjyxcpnL7emi7SruQ98eP7mC8cDNkjCKfXNZ7
  Account ID:        0x182a8385912e206910a11dfcfbadcc018d73ff3febe36f9eb85774e77591a314
  SS58 Address:      5CcPdpUAhRvGjyxcpnL7emi7SruQ98eP7mC8cDNkjCKfXNZ7
```


You can also create a vanity address, meaning an address that contains a specified sub-string. But
you will not receive a mnemonic phrase for this address.

_Command:_

```bash
subkey vanity --pattern dan
```

_Output:_

```text
Generating key containing pattern 'dan'
best: 189 == top: 189
Secret Key URI `0xf58706b2057633b6d634d9cc54e8e9e8f11246290c0bbef934129978ede6439b` is account:
  Secret seed:       0xf58706b2057633b6d634d9cc54e8e9e8f11246290c0bbef934129978ede6439b
  Public key (hex):  0xca043bd23428eb023d59f4b7eb37416b343fc719a3c89c5256071b4f39d73130
  Public key (SS58): 5GdandXEVW1ER8cxW6uQgpjpyuEbZ3s2ED3ZU15LfrGdQhhz
  Account ID:        0xca043bd23428eb023d59f4b7eb37416b343fc719a3c89c5256071b4f39d73130
  SS58 Address:      5GdandXEVW1ER8cxW6uQgpjpyuEbZ3s2ED3ZU15LfrGdQhhz
```

Notice the SS58 Address`5GdandXEVW1ER8cxW6uQgpjpyuEbZ3s2ED3ZU15LfrGdQhhz` contains the string
`dan`.

### Example seed

> We will use the seed generated above to illustrate different
> [HD key derivation paths](#hd-key-derivation) and
> [passwords](#password-protected-keys):
>
> ```text
> caution juice atom organ advance problem want pledge someone senior holiday very
> ```

_Command:_

```bash
subkey inspect "caution juice atom organ advance problem want pledge someone senior holiday very"
```

_Output:_

```text
Secret phrase `caution juice atom organ advance problem want pledge someone senior holiday very` is account:
  Secret seed:       0xc8fa03532fb22ee1f7f6908b9c02b4e72483f0dbd66e4cd456b8f34c6230b849
  Public key (hex):  0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  Public key (SS58): 5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
  Account ID:        0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  SS58 Address:      5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
```

## Specify network address format

Subkey encodes the address depending on the network. You can use the `--network` flag to change
this. See `subkey <inspect/generate> -n` for supported networks.

Let's say you want to use the **same private key** on the Kusama network. Use `--network` to get
your **address** formatted for Kusama. Notice that the public key is the same, but the address has
a different format. Here we use the [example seed](#example-seed):

_Command:_

```bash
subkey inspect --network kusama "caution juice atom organ advance problem want pledge someone senior holiday very"
```

_Output:_

```text
Secret phrase `caution juice atom organ advance problem want pledge someone senior holiday very` is account:
  Secret seed:       0xc8fa03532fb22ee1f7f6908b9c02b4e72483f0dbd66e4cd456b8f34c6230b849
  Public key (hex):  0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  Public key (SS58): HRkCrbmke2XeabJ5fxJdgXWpBRPkXWfWHY8eTeCKwDdf4k6
  Account ID:        0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  SS58 Address:      HRkCrbmke2XeabJ5fxJdgXWpBRPkXWfWHY8eTeCKwDdf4k6
```

## HD key derivation

<Message
  type={`yellow`}
  title={`Important`}
  text={`Remember that passwords and derivation paths are just as important to backup as the seed itself!
  See the [password note](#important-password-and-derivation-note) for more information.`}
/>

Subkey supports hard and soft hierarchical deterministic (HD) key derivation compliant with
[BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki). HD keys allow you to have a
master seed and define a tree with a key pair at each leaf. The tree has a similar structure to a
filesystem and can have any depth you want.

Hard and soft key derivation both support:

- parent private key + path ➔ child private key
- parent private key + path ➔ child public key

Further, soft key derivation supports:

- parent public key + path ➔ child public key

### Hard key derivation

You can derive a hard key child using `//` after the mnemonic phrase:

_Command:_

```bash
subkey inspect "caution juice atom organ advance problem want pledge someone senior holiday very//polkadot//0"
```

_Output:_

```text
Secret Key URI `caution juice atom organ advance problem want pledge someone senior holiday very//polkadot//0` is account:
  Secret seed:       0x056a6a4e203766ffbea3146967ef25e9daf677b14dc6f6ed8919b1983c9bebbc
  Public key (hex):  0x841226ea070c9577979ca2e854130fbe3253853c13c05943e09908312950275d
  Public key (SS58): 5F3sa2TJAWMqDhXG6jhV4N8ko9SxwGy8TpaNS1repo5EYjQX
  Account ID:        0x841226ea070c9577979ca2e854130fbe3253853c13c05943e09908312950275d
  SS58 Address:      5F3sa2TJAWMqDhXG6jhV4N8ko9SxwGy8TpaNS1repo5EYjQX
```

### Soft key derivation

Likewise, you can derive a soft key child using a single `/` after the mnemonic phrase:

_Command:_

```bash
subkey inspect "caution juice atom organ advance problem want pledge someone senior holiday very/polkadot/0"
```

_Output:_

```text
Secret Key URI `caution juice atom organ advance problem want pledge someone senior holiday very/polkadot/0` is account:
  Secret seed:       n/a
  Public key (hex):  0x94bec5342b23d74b8736c74ab1bf311182c62510d220313a424e63ea57172855
  Public key (SS58): 5FRjccB7s9fbMu4pwQhYng2quQnKYkCHXRUCMBRwL7Pzj8FX
  Account ID:        0x94bec5342b23d74b8736c74ab1bf311182c62510d220313a424e63ea57172855
  SS58 Address:      5FRjccB7s9fbMu4pwQhYng2quQnKYkCHXRUCMBRwL7Pzj8FX
```

Recall the _root_ SS58 address from the [example seed](#example-seed) is,
`5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR`. We can use that _public address_ to derive the
same child address as above using the _seed_.

_Command:_

```bash
subkey inspect "5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR/polkadot/0"
```

_Output:_

```text
Public Key URI `5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR/polkadot/0` is account:
  Network ID/version: substrate
  Public key (hex):   0x94bec5342b23d74b8736c74ab1bf311182c62510d220313a424e63ea57172855
  Public key (SS58):  5FRjccB7s9fbMu4pwQhYng2quQnKYkCHXRUCMBRwL7Pzj8FX
  Account ID:         0x94bec5342b23d74b8736c74ab1bf311182c62510d220313a424e63ea57172855
  SS58 Address:       5FRjccB7s9fbMu4pwQhYng2quQnKYkCHXRUCMBRwL7Pzj8FX
```

Note that the two addresses here match. This is not the case in hard key derivation!

## Password protected keys

<Message
  type={`yellow`}
  title={`Important`}
  text={`Remember that passwords and derivation paths are just as important to backup as the seed itself!
  See the [password note](#important-password-and-derivation-note) for more information.`}
/>

To generate a key that is password protected, use the `--password <password>` flag:

_Command:_

```bash
subkey generate --password "pencil laptop kitchen cutter"
```

_Output:_

```text
Secret phrase `apology truck manage valve merge front idea imitate coconut poverty trap gas` is account:
  Secret seed:       0x2da5ddf709c8cd6c9727d347cd1e867234802d15ce51581d86bcb949bba7bcb5
  Public key (hex):  0xc8e06268df6173a32a859da4b9601d5e11b503206f9817ae1c101d2984826d19
  Public key (SS58): 5Gc66BfkkebMv3EvP8ThfpiGLng1EPtUiAQ71gzwpJosJqvo
  Account ID:        0xc8e06268df6173a32a859da4b9601d5e11b503206f9817ae1c101d2984826d19
  SS58 Address:      5Gc66BfkkebMv3EvP8ThfpiGLng1EPtUiAQ71gzwpJosJqvo
```

You can inspect password-protected keys either by passing the `--password` flag or using `///` at
the end of the mnemonic:

_Command (--password flag):_

```bash
subkey inspect "caution juice atom organ advance problem want pledge someone senior holiday very" --password "pencil laptop kitchen cutter"
```

_Output (--password flag):_

```text
Secret phrase `caution juice atom organ advance problem want pledge someone senior holiday very` is account:
  Secret seed:       0xdfc5d5d5235a37fdc907ee1cb720299f96aeb02f9c7c2fcad7ee8c7bfbd2a4db
  Public key (hex):  0xdef8f78b123475265815b65a7c55e105e1ab185f4969954f68d92b7bb67a1045
  Public key (SS58): 5H74SqH1iQCWh5Gumyghh1WJMcmM6TdBHYSK7mKVJbv9NuSK
  Account ID:        0xdef8f78b123475265815b65a7c55e105e1ab185f4969954f68d92b7bb67a1045
  SS58 Address:      5H74SqH1iQCWh5Gumyghh1WJMcmM6TdBHYSK7mKVJbv9NuSK
```

_Command (password with `///`):_

```bash
subkey inspect "caution juice atom organ advance problem want pledge someone senior holiday very///pencil laptop kitchen cutter"
```

_Output (password with `///`):_

```text
Secret Key URI `caution juice atom organ advance problem want pledge someone senior holiday very///pencil laptop kitchen cutter` is account:
  Secret seed:       0xdfc5d5d5235a37fdc907ee1cb720299f96aeb02f9c7c2fcad7ee8c7bfbd2a4db
  Public key (hex):  0xdef8f78b123475265815b65a7c55e105e1ab185f4969954f68d92b7bb67a1045
  Public key (SS58): 5H74SqH1iQCWh5Gumyghh1WJMcmM6TdBHYSK7mKVJbv9NuSK
  Account ID:        0xdef8f78b123475265815b65a7c55e105e1ab185f4969954f68d92b7bb67a1045
  SS58 Address:      5H74SqH1iQCWh5Gumyghh1WJMcmM6TdBHYSK7mKVJbv9NuSK
```

Take note that the `--password` flag _does not_ report the password used, but the `///` _does_.
The output is identical otherwise.

## Combine derivation paths and passwords

<Message
  type={`yellow`}
  title={`Important password and derivation note`}
  text={`Note that the "secret seed" _is not_ password protected. Although it can still recover _an_
account, the key pair that's derived _is not the same_ account as recovered with _any_ password!
The same seed with different derivation paths passwords will derive **different keys**!
**The "Secret phrase" is not sufficient to recover a key pair!**
Keep your paths and passwords secure, as without it your key pair cannot be recovered!`}
/>

You can mix and match hard and soft key paths (although it doesn't make much sense to have hard
paths as children of soft paths). For example:

_Command:_

```bash
subkey inspect "caution juice atom organ advance problem want pledge someone senior holiday very//polkadot/0"
```

_Output:_

```text
Secret Key URI `caution juice atom organ advance problem want pledge someone senior holiday very//polkadot/0` is account:
  Secret seed:       n/a
  Public key (hex):  0xd6f066a07b3ebb1572c00506c9dc86a64db63779e7791e7d37247f59e4bffc14
  Public key (SS58): 5GvXX2NpiR8qoeTUbdzTCmMtZgGrZxRoCJFEvfL1iaDjTFKb
  Account ID:        0xd6f066a07b3ebb1572c00506c9dc86a64db63779e7791e7d37247f59e4bffc14
  SS58 Address:      5GvXX2NpiR8qoeTUbdzTCmMtZgGrZxRoCJFEvfL1iaDjTFKb
```

The first level (`//polkadot`) is **hard-derived**, while the leaf (`/0`) is **soft-derived**.

To use key derivation with a password-protected key, add your password to the end:

_Command:_

```bash
# Mnemonic phrase + derivation path + password
subkey inspect "caution juice atom organ advance problem want pledge someone senior holiday very//polkadot/0///pencil laptop kitchen cutter"
```

_Output:_

```text
Secret Key URI `caution juice atom organ advance problem want pledge someone senior holiday very//polkadot/0///pencil laptop kitchen cutter` is account:
  Secret seed:       n/a
  Public key (hex):  0xcaf1f5ec14b507b5c365c6528cc06de74a5615274694a0c895ed4109c0ff0d32
  Public key (SS58): 5GeoQa3nkeNmzZSfgBFuK3BkAggnTHcX3S1j94sffJYYphrP
  Account ID:        0xcaf1f5ec14b507b5c365c6528cc06de74a5615274694a0c895ed4109c0ff0d32
  SS58 Address:      5GeoQa3nkeNmzZSfgBFuK3BkAggnTHcX3S1j94sffJYYphrP
```

Notice that for **soft-derived** keys, you can use the hard-derived (with optional password)
**parent's public address** to derive new public addresses:

_Command:_

```bash
# A soft-derived SS58-address for a *public address* with a hidden seed, hard derivation path, and password
subkey inspect "5GsbzysSK8TKahXBC7FpS2myx3nWehMyYU7q8CLrzZCjpKbM/0"
```

_Output:_

```text
Public Key URI `5GsbzysSK8TKahXBC7FpS2myx3nWehMyYU7q8CLrzZCjpKbM/0` is account:
  Network ID/version: substrate
  Public key (hex):   0xcaf1f5ec14b507b5c365c6528cc06de74a5615274694a0c895ed4109c0ff0d32
  Public key (SS58):  5GeoQa3nkeNmzZSfgBFuK3BkAggnTHcX3S1j94sffJYYphrP
  Account ID:         0xcaf1f5ec14b507b5c365c6528cc06de74a5615274694a0c895ed4109c0ff0d32
  SS58 Address:       5GeoQa3nkeNmzZSfgBFuK3BkAggnTHcX3S1j94sffJYYphrP
```

Notice that the "SS58-address + derivation path" produces the same address as the "mnemonic
phrase + derivation path + password." As such, you can reveal your parent public address
and soft derivation paths without revealing your mnemonic phrase or password, retaining control
of all derived addresses.

## Signing and verifying messages

### Signing

You can sign a message by passing the message to Subkey on STDIN. You can sign with either your seed
or mnemonic phrase. We use the [example raw seed and address](#example-seed) here:

_Command:_

```bash
# Usage
# echo "msg" | subkey sign --suri <secret-seed>

# Example
echo "test message" | subkey sign --suri 0xc8fa03532fb22ee1f7f6908b9c02b4e72483f0dbd66e4cd456b8f34c6230b849
```

_Output:_

```text
22f91b41ba12f8663ddce26bfc90dbfe6a51683fd3782ad679ab2a5fdbe7d44c2a119f22c74eea22555e5483eb7f42b828f189a38379d59c3b607d2461f0858e
```

### Verifying

Although the signatures above are different, they both verify with the address:

_Command:_

```bash
# Usage
# echo "msg" | subkey verify <signature> <SS58-address>

# Example
echo "test message" | subkey verify 22f91b41ba12f8663ddce26bfc90dbfe6a51683fd3782ad679ab2a5fdbe7d44c2a119f22c74eea22555e5483eb7f42b828f189a38379d59c3b607d2461f0858e 5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
```

_Output:_

```text
Signature verifies correctly.
```

<br />
<Message
  type={`gray`}
  title={`Note`}
  text={`\`echo\` appends a newline character to the end of strings. Therefore, if you verify a message
  that was signed elsewhere, e.g. Polkadot JS, then you will need to use \`echo -n\` to remove the
  newline character and verify the correct message.`}
/>

## Generating node keys

You can generate a node's libp2p key by the following:

_Command:_

```bash
# Usage
# subkey generate-node-key --file <output-file>

# Example
subkey generate-node-key --file node-key
```

_Output:_

```text
12D3KooWAhzqC4xstZKkAVYge2Q9tAMcnFhNEabLejRNVoQWLHHC
```

The peer ID is displayed on screen and the actual key is saved in the `<output-file>`.

## Well-known keys

The common seed used for all development accounts is:

```text
bottom drive obey lake curtain smoke basket hold race lonely fit walk
```

Various development accounts are _derived_ from this seed.

If you've worked with Substrate previously, you have likely encountered the ubiquitous accounts for
Alice, Bob, and their friends. These keys are not at all private, but are useful for playing with
Substrate without always generating new key pairs. You can inspect these "well-known" keys as well.

_Command:_

```bash
subkey inspect //Alice
```

_Output:_

```text
Secret Key URI `//Alice` is account:
  Secret seed:      0xe5be9a5092b81bca64be81d212e7f2f9eba183bb7a90954f7b76361f6edb5c0a
  Public key (hex): 0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
  Account ID:       0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
  SS58 Address:     5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
```

A mnemonic phrase is a secret group of words that can be used to uniquely generate a private key. In
Subkey, there is a specific dev phrase used whenever a mnemonic phrase is omitted.`//Alice` for
example is actually:

```text
bottom drive obey lake curtain smoke basket hold race lonely fit walk//Alice
```

It is important remember that keys such as this are _not_ secure, and should be deployed only for
testing. In addition, anyone not using PolkadotJS or Subkey should note this information to properly
generate the account.

## Further resources

<Message
  type={`yellow`}
  title={`Important`}
  text={`Remember that passwords and derivation paths are just as important to backup as the seed itself!
  See the [password note](#important-password-and-derivation-note) for more information.`}
/>

- Learn more by running `subkey help` or see the
  [README](https://github.com/paritytech/substrate/tree/master/bin/utils/subkey).
- Key pairs can also be generated in the [PolkadotJS Apps UI](https://polkadot.js.org/apps/). Try
  creating keys with the UI and restoring them with Subkey or vice versa.
- Learn more about
  [different cryptographic algorithms and choosing between them](https://wiki.polkadot.network/docs/en/learn-cryptography).
