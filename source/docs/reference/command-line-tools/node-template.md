A simple CLI tool to launch a local Polkadot test network.
[Go to Documentation](https://github.com/paritytech/polkadot-launch).

# node-template

A CLI tool to interact with any blockchain binary built with Substrate. 

- What is it and what you can use it for?
- Background info if relevant

This CLI tool comes with any Substrate blockchain built using the [Substrate node template](https://github.com/substrate-developer-hub/substrate-node-template), suited for users running the binary of their chain.
This implies the tool can be used on the binary of any Substrate chain.
This reference manual assumes that users are running the CLI tool on the Substrate node template. 
However, it applies for any chain.
## Installation 

To use the CLI:

1. Clone the [Substrate node template](https://github.com/substrate-developer-hub/substrate-node-template) and build it by running `cargo build --release`. Skip this step if you're already working with a compiled chain.

1. Go to the `target/release` folder of your chain: `cd substrate-node-template/target/release`.

1. Run `./node-template --help` to view the different commands and sub-commands. 
   Replace `node-template` with your chain's binary if you are using the CLI on your own chain. 

## Basic command usage

The basic syntax for running `node-template` commands is:

`./target/release/node-template [subcommand] [flag]`

### Flags

You can use the following optional flags with the `node-template` command.

| Flag   | Description
| ------ | -----------
| -h, --help | Displays usage information.
| -V, --version | Displays the version of the chain's binary.

### Subcommands

You can use the following subcommands with the `node-template` command. 
For reference information and examples that illustrate using {`tool`} subcommands, select an appropriate command.

| Command | Description
| ------- | -----------
| `benchmark` | Benchmark runtime pallets.
| `build-spec` | Build a chain specification.
| `check-block` | Validate blocks.
| `export-blocks` | Export blocks.
| `export-state` | Export the state of a given block into a chain spec.
| `help` | Prints this message or the help of the given subcommand(s).
| `import-blocks` | Import blocks.
| `key` | Key management CLI utilities.
| `purge-chain` | Remove the whole chain.
| `revert` | Revert the chain to a previous state.
|

### Output

Depending on the subcommand you specify, the output from the {`tool`} program displays some or all of the following information:

| This field | Contains
| ---------- | ----------

### Examples

To {_describe some use_}, run the following command:

`tool --use`

## {specific subcommand}

Use the {tool subcommand} command to {_what the subcommand does_}.

#### Basic usage

`tool subcommand [FLAGS] [OPTIONS] <signature> <uri>`

#### Flags

You can use the following optional flags with the {`tool subcommand`} command.

| Flag   | Description
| ------ | -----------
| `--alice` | Shortcut for `--name Alice --validator` with session keys for `Alice` added to keystore.
| `--allow-private-ipv4` | Always accept connecting to private IPv4 addresses (as specified in [RFC1918](https://tools.ietf.org/html/rfc1918). Enabled by default for chains marked as "local" in their chain specifications, or when `--dev` is passed.
| `--bob` | Shortcut for `--name Bob --validator` with session keys for `Bob` added to keystore.
| ` --charlie` |  Shortcut for `--name Charlie --validator` with session keys for `Charlie` added to keystore.
| `--dave` | Shortcut for `--name Dave --validator` with session keys for `Dave` added to keystore.
| `--detailed-log-output` | Enable detailed log output. This includes displaying the log target, log level and thread name. This is automatically enabled when something is logged with any higher level than `info`.
| `--dev` | Specify the development chain. This flag sets `--chain=dev`, `--force-authoring`, `--rpc-cors=all`, `--alice`, and `--tmp` flags, unless explicitly overridden.
| `--disable-log-color` |  Disable log color output.
| `--discover-local` | Enable peer discovery on local networks. By default this option is `true` for `--dev` or when the chain type is `Local`/`Development` and false otherwise.
| `--enable-log-reloading` | Enable feature to dynamically update and reload the log filter. Be aware that enabling this feature can lead to a performance decrease up to factor six or more. Depending on the global logging level the performance decrease changes. The `system_addLogFilter` and `system_resetLogFilter` RPCs will have no effect with this option not being set.
| `--eve` | Shortcut for `--name Eve --validator` with session keys for `Eve` added to keystore.
| `--ferdie` | Shortcut for `--name Ferdie --validator` with session keys for `Ferdie` added to keystore.
| `--force-authoring` | Enable authoring even when offline.
| `-h, --help` | Prints help information.
| `--ipfs-server` | Join the IPFS network and serve transactions over bitswap protocol.
| `--kademlia-disjoint-query-paths` | Require iterative Kademlia DHT queries to use disjoint paths for increased resiliency in the presence of potentially adversarial nodes. See the S/Kademlia paper for more information on the high level design as well as its security improvements.
| `--light` | Experimental: Run in light client mode.
| `--no-grandpa` | Disable GRANDPA voter when running in validator mode, otherwise disable the GRANDPA observer. 
| `--no-mdns` | Disable mDNS discovery. By default, the network will use mDNS to discover other nodes on the local network. This disables it. Automatically implied when using `--dev`.
| `--no-private-ipv4` | Always forbid connecting to private IPv4 addresses (as specified in [RFC1918](https://tools.ietf.org/html/rfc1918)), unless the address was passed with `--reserved-nodes` or `--bootnodes`. Enabled by default for chains marked as "live" in their chain specifications.
| `--no-prometheus` | Do not expose a Prometheus exporter endpoint. Prometheus metric endpoint is enabled by default.
| `--no-telemetry` | Disable connecting to the Substrate telemetry server. Telemetry is on by default on global chains.
| `--one` | Shortcut for `--name One --validator` with session keys for `One` added to keystore.
| `--password-interactive` | Use interactive shell for entering the password used by the keystore.
| `--prometheus-external` | Expose Prometheus exporter on all interfaces. Default is local.
| `--reserved-only` | Whether to only synchronize the chain with reserved nodes. Also disables automatic peer discovery. TCP connections might still be established with non-reserved nodes. In particular, if you are a validator your node might still connect to other validator nodes and collator nodes regardless of whether they are defined as reserved nodes.
| `--rpc-external` | Listen to all RPC interfaces. Default is local. Note: not all RPC methods are safe to be exposed publicly. Use an RPC proxy server to filter out dangerous methods. More details: <https://docs.substrate.io/v3/runtime/custom-rpcs/#public-rpcs>. Use `--unsafe-rpc-external` to suppress the warning if you understand the risks.
| `--storage-chain` | Enable storage chain mode. This changes the storage format for blocks bodies. If this is enabled, each transaction is stored separately in the transaction database column and is only referenced by hash in the block body column.
| `--tmp` | Run a temporary node. A temporary directory will be created to store the configuration and will be deleted at the end of the process. Note: the directory is random per process execution. This directory is used as base path which includes: database, node key and keystore. When `--dev` is given and no explicit `--base-path`, this option is implied.
| `--two` | Shortcut for `--name Two --validator` with session keys for `Two` added to keystore. 
| `--unsafe-pruning` | Force start with unsafe pruning settings. When running as a validator it is highly recommended to disable state pruning (i.e. 'archive') which is the default. The node will refuse to start as a validator if pruning is enabled unless this option is set.
| `--unsafe-rpc-external` | Listen to all RPC interfaces. Same as `--rpc-external`.
| `--unsafe-ws-external` | Listen to all Websocket interfaces. Same as `--ws-external` but doesn't warn you about it.
| `--validator` | Enable validator mode. The node will be started with the authority role and actively participate in any consensus task that it can (e.g. depending on availability of local keys).
| `-V, --version` | Prints version information.
| `--ws-external` | Listen to all Websocket interfaces. Default is local. Note: not all RPC methods are safe to be exposed publicly. Use an RPC proxy server to filter out dangerous methods. More details: <https://docs.substrate.io/v3/runtime/custom-rpcs/#public-rpcs>. Use `--unsafe-ws-external` to suppress the warning if you understand the risks.


#### Options

You can use the following command-line options with the {`tool subcommand`} command.

| Option   | Description
| -------- | -----------
|          

#### Examples

{2-3 examples on ways to use this subcommand and any relevant explanations}