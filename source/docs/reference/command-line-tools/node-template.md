# node-template

The `node-template` program provides a working Substrate node with FRAME system pallets and a subset of additional pallets for working with common blockchain functional operations.`
With its baseline of functional pallets, the `node-template` serves as a starter kit for building your own blockchain and developing a custom runtime.
You can use the `node-template` program to perform the following tasks:

## Basic command usage

The basic syntax for running `subkey` commands is:

```
node-template [subcommand] [FLAGS] [OPTIONS]
```

Depending on the subcommand you specify, additional arguments, options, and flags might apply or be required. 
To view usage information for a specific `node-template` subcommand, specify the subcommand and the `--help` flag. 
For example, to see usage information for `node-template key`, you can run the following command:

```
node-template key --help
```

### Flags

You can use the following optional flags with the `node-template` command.

| Flag   | Description
| ------ | -----------
| `--alice`  | Adds the session keys for the predefined `Alice` account to the local keystore. This flag is equivalent to running the node using `--name alice --validator` as command-line options.
| `--allow-private-ipv4` | Allows the node to connect to private IPv4 addresses (as specified in [RFC1918](https://tools.ietf.org/html/rfc1918)). This flag is enabled by default if the chain specifications for the node is identified as `local` or you start the node in development mode with the `--dev` flag.
| `--bob` | Adds the session keys for the predefined `Bob` account to the local keystore. This flag is equivalent to running the node using `--name bob --validator` as command-line options.
| `--charlie` | Adds the session keys for the predefined `Charlie` account to the local keystore. This flag is equivalent to running the node using `--name charlie --validator` as command-line options.
| `--dave` | Adds the session keys for the predefined `Dave` account to the local keystore. This flag is equivalent to running the node using `--name dave --validator` as command-line options.
| `--dev` | Starts the node in development mode in a fresh state. No state is persisted if you run the node using this flag.
| `--disable-log-color` | Disables the use of color in log messages.
| `--disable-log-reloading` | Disables log filter updates and reloading. By default, dynamic log filtering is enabled. However, the feature can affect performance. If you start the node with this flag, the `system_addLogFilter` and `system_resetLogFilter` remote procedure calls have no effect.
| `--discover-local`| Enables peer discovery on local networks. By default, this flag is `true` if you start the node using the `--dev` flag or if the chain specification is `Local` or `Development` and `false` otherwise.
| `--eve` | Adds the session keys for the predefined `Eve` account to the local keystore. This flag is equivalent to running the node using `--name eve --validator` as command-line options.
| `--ferdie` | Adds the session keys for the predefined `Ferdie` account to the local keystore. This flag is equivalent to running the node using `--name ferdie --validator` as command-line options.
| `--force-authoring` | Enables block authoring even if the node is offline.
| `--ipfs-server` | Joins the IPFS network and serve transactions over bitswap protocol
| `--kademlia-disjoint-query-paths` | Requires iterative Kademlia distributed hash table (DHT) queries to use disjointed paths. This option increases resiliency in the presence of potentially adversarial nodes. See the S/Kademlia paper for more information on the high level design as well as its security improvements.
| `--light` | Runs the node in light client mode (experimental).
| `--no-grandpa` | Disables the GRANDPA voter if the node is running as a validator mode.If the node is not running as a validator, the option disables the GRANDPA observer.
| `-h`, `--help` | Displays usage information.
| `-V`, `--version` | Displays version information.

FLAGS:


--no-mdns | Disable mDNS discovery. By default, the network will use mDNS to discover other nodes on the local network. This disables it. Automatically implied when using --dev.
--no-private-ipv4 | Always forbid connecting to private IPv4 addresses (as specified in [RFC1918](https://tools.ietf.org/html/rfc1918)), unless the address was passed with `--reserved-nodes` or `--bootnodes`. Enabled by default for chains marked as "live" in their chain specifications
--no-prometheus | Do not expose a Prometheus exporter endpoint. Prometheus metric endpoint is enabled by default.
        --no-telemetry                     
            Disable connecting to the Substrate telemetry server.
            
            Telemetry is on by default on global chains.
        --one                              
            Shortcut for `--name One --validator` with session keys for `One` added to keystore

        --password-interactive             
            Use interactive shell for entering the password used by the keystore

        --prometheus-external              
            Expose Prometheus exporter on all interfaces.
            
            Default is local.
        --reserved-only                    
            Whether to only synchronize the chain with reserved nodes.
            
            Also disables automatic peer discovery.
            
            TCP connections might still be established with non-reserved nodes. In particular, if you are a validator
            your node might still connect to other validator nodes and collator nodes regardless of whether they are
            defined as reserved nodes.
        --rpc-external                     
            Listen to all RPC interfaces.
            
            Default is local. Note: not all RPC methods are safe to be exposed publicly. Use an RPC proxy server to
            filter out dangerous methods. More details: <https://github.com/paritytech/substrate/wiki/Public-RPC>. Use
            `--unsafe-rpc-external` to suppress the warning if you understand the risks.
        --storage-chain                    
            Enable storage chain mode
            
            This changes the storage format for blocks bodies. If this is enabled, each transaction is stored separately
            in the transaction database column and is only referenced by hash in the block body column.
        --tmp                              
            Run a temporary node.
            
            A temporary directory will be created to store the configuration and will be deleted at the end of the
            process.
            
            Note: the directory is random per process execution. This directory is used as base path which includes:
            database, node key and keystore.
        --two                              
            Shortcut for `--name Two --validator` with session keys for `Two` added to keystore

        --unsafe-pruning                   
            Force start with unsafe pruning settings.
            
            When running as a validator it is highly recommended to disable state pruning (i.e. 'archive') which is the
            default. The node will refuse to start as a validator if pruning is enabled unless this option is set.
        --unsafe-rpc-external              
            Listen to all RPC interfaces.
            
            Same as `--rpc-external`.
        --unsafe-ws-external               
            Listen to all Websocket interfaces.
            
            Same as `--ws-external` but doesn't warn you about it.
        --validator                        
            Enable validator mode.
            
            The node will be started with the authority role and actively participate in any consensus task that it can
            (e.g. depending on availability of local keys).
    -V, --version                          
            Prints version information

        --ws-external                      
            Listen to all Websocket interfaces.
            
            Default is local. Note: not all RPC methods are safe to be exposed publicly. Use an RPC proxy server to
            filter out dangerous methods. More details: <https://github.com/paritytech/substrate/wiki/Public-RPC>. Use
            `--unsafe-ws-external` to suppress the warning if you understand the risks.

OPTIONS:
    -d, --base-path <PATH>                                       
            Specify custom base path

        --bootnodes <ADDR>...                                    
            Specify a list of bootnodes

        --chain <CHAIN_SPEC>                                     
            Specify the chain specification.
            
            It can be one of the predefined ones (dev, local, or staging) or it can be a path to a file with the
            chainspec (such as one exported by the `build-spec` subcommand).
        --database <DB>
            Select database backend to use [possible values: rocksdb, paritydb-experimental]

        --db-cache <MiB>                                         
            Limit the memory the database cache can use

        --offchain-worker <ENABLED>
            Should execute offchain workers on every block.
            
            By default it's only enabled for nodes that are authoring new blocks. [default: WhenValidating]
            [possible values: Always, Never, WhenValidating]
        --execution <STRATEGY>
            The execution strategy that should be used by all execution contexts [possible values: Native,
            Wasm, Both, NativeElseWasm]
        --execution-block-construction <STRATEGY>
            The means of execution used when calling into the runtime while constructing blocks [possible values:
            Native, Wasm, Both, NativeElseWasm]
        --execution-import-block <STRATEGY>
            The means of execution used when calling into the runtime for general block import (including locally
            authored blocks) [possible values: Native, Wasm, Both, NativeElseWasm]
        --execution-offchain-worker <STRATEGY>
            The means of execution used when calling into the runtime while using an off-chain worker [possible values:
            Native, Wasm, Both, NativeElseWasm]
        --execution-other <STRATEGY>
            The means of execution used when calling into the runtime while not syncing, importing or constructing
            blocks [possible values: Native, Wasm, Both, NativeElseWasm]
        --execution-syncing <STRATEGY>
            The means of execution used when calling into the runtime for importing blocks as part of an initial sync
            [possible values: Native, Wasm, Both, NativeElseWasm]
        --in-peers <COUNT>
            Specify the maximum number of incoming connections we're accepting [default: 25]

        --enable-offchain-indexing <ENABLE_OFFCHAIN_INDEXING>
            Enable Offchain Indexing API, which allows block import to write to Offchain DB.
            
            Enables a runtime to write directly to a offchain workers DB during block import.
        --ipc-path <PATH>                                        
            Specify IPC RPC server path

        --keep-blocks <COUNT>
            Specify the number of finalized blocks to keep in the database.
            
            Default is to keep all blocks.
        --keystore-path <PATH>                                   
            Specify custom keystore path

        --keystore-uri <keystore-uri>                            
            Specify custom URIs to connect to for keystore-services

        --listen-addr <LISTEN_ADDR>...                           
            Listen on this multiaddress.
            
            By default: If `--validator` is passed: `/ip4/0.0.0.0/tcp/<port>` and `/ip6/[::]/tcp/<port>`. Otherwise:
            `/ip4/0.0.0.0/tcp/<port>/ws` and `/ip6/[::]/tcp/<port>/ws`.
    -l, --log <LOG_PATTERN>...
            Sets a custom logging filter. Syntax is <target>=<level>, e.g. -lsync=debug.
            
            Log levels (least to most verbose) are error, warn, info, debug, and trace. By default, all targets log
            `info`. The global log level can be set with -l<level>.
        --max-parallel-downloads <COUNT>
            Maximum number of peers from which to ask for the same blocks in parallel.
            
            This allows downloading announced blocks from multiple peers. Decrease to save traffic and risk increased
            latency. [default: 5]
        --max-runtime-instances <max-runtime-instances>          
            The size of the instances cache for each runtime.
            
            The default value is 8 and the values higher than 256 are ignored.
        --name <NAME>                                            
            The human-readable name for this node.
            
            The node name will be reported to the telemetry server, if enabled.
        --node-key <KEY>                                         
            The secret key to use for libp2p networking.
            
            The value is a string that is parsed according to the choice of `--node-key-type` as follows:
            
            `ed25519`: The value is parsed as a hex-encoded Ed25519 32 byte secret key, i.e. 64 hex characters.
            
            The value of this option takes precedence over `--node-key-file`.
            
            WARNING: Secrets provided as command-line arguments are easily exposed. Use of this option should be limited
            to development and testing. To use an externally managed secret key, use `--node-key-file` instead.
        --node-key-file <FILE>
            The file from which to read the node's secret key to use for libp2p networking.
            
            The contents of the file are parsed according to the choice of `--node-key-type` as follows:
            
            `ed25519`: The file must contain an unencoded 32 byte or hex encoded Ed25519 secret key.
            
            If the file does not exist, it is created with a newly generated secret key of the chosen type.
        --node-key-type <TYPE>
            The type of secret key to use for libp2p networking.
            
            The secret key of the node is obtained as follows:
            
            * If the `--node-key` option is given, the value is parsed as a secret key according to the type. See the
            documentation for `--node-key`.
            
            * If the `--node-key-file` option is given, the secret key is read from the specified file. See the
            documentation for `--node-key-file`.
            
            * Otherwise, the secret key is read from a file with a predetermined, type-specific name from the chain-
            specific network config directory inside the base directory specified by `--base-dir`. If this file
            does not exist, it is created with a newly generated secret key of the chosen type.
            
            The node's secret key determines the corresponding public key and hence the node's peer ID in the context of
            libp2p. [default: Ed25519]  [possible values: Ed25519]
        --out-peers <COUNT>
            Specify the number of outgoing connections we're trying to maintain [default: 25]

        --password <password>                                    
            Password used by the keystore

        --password-filename <PATH>                               
            File that contains the password used by the keystore

        --pool-kbytes <COUNT>
            Maximum number of kilobytes of all transactions stored in the pool [default: 20480]

        --pool-limit <COUNT>
            Maximum number of transactions in the transaction pool [default: 8192]

        --port <PORT>                                            
            Specify p2p protocol TCP port

        --prometheus-port <PORT>                                 
            Specify Prometheus exporter TCP Port

        --pruning <PRUNING_MODE>
            Specify the state pruning mode, a number of blocks to keep or 'archive'.
            
            Default is to keep all block states if the node is running as a validator (i.e. 'archive'), otherwise state
            is only kept for the last 256 blocks.
        --public-addr <PUBLIC_ADDR>...
            The public address that other nodes will use to connect to it. This can be used if there's a proxy in front
            of this node
        --reserved-nodes <ADDR>...                               
            Specify a list of reserved node addresses

        --rpc-cors <ORIGINS>
            Specify browser Origins allowed to access the HTTP & WS RPC servers.
            
            A comma-separated list of origins (protocol://domain or special `null` value). Value of `all` will disable
            origin validation. Default is to allow localhost and <https://polkadot.js.org> origins. When running in
            --dev mode the default is to allow all origins.
        --rpc-http-threads <COUNT>                               
            Size of the RPC HTTP server thread pool

        --rpc-max-payload <rpc-max-payload>
            Set the the maximum RPC payload size for both requests and responses (both http and ws), in megabytes.
            Default is 15MiB
        --rpc-methods <METHOD SET>
            RPC methods to expose.
            
            - `Unsafe`: Exposes every RPC method.
            - `Safe`: Exposes only a safe subset of RPC methods, denying unsafe RPC methods.
            - `Auto`: Acts as `Safe` if RPC is served externally, e.g. when `--{rpc,ws}-external` is
              passed, otherwise acts as `Unsafe`. [default: Auto]  [possible values: Auto, Safe,
            Unsafe]
        --rpc-port <PORT>                                        
            Specify HTTP RPC server TCP port

        --state-cache-size <Bytes>                               
            Specify the state cache size [default: 67108864]

        --sync <SYNC_MODE>                                       
            Blockchain syncing mode.
            
            - `Full`: Download and validate full blockchain history.
            
            - `Fast`: Download blocks and the latest state only.
            
            - `FastUnsafe`: Same as `Fast`, but skip downloading state proofs. [default: Full]
        --telemetry-url <URL VERBOSITY>...                       
            The URL of the telemetry server to connect to.
            
            This flag can be passed multiple times as a means to specify multiple telemetry endpoints. Verbosity levels
            range from 0-9, with 0 denoting the least verbosity. Expected format is 'URL VERBOSITY', e.g. `--telemetry-
            url 'wss://foo/bar 0'`.
        --tracing-receiver <RECEIVER>
            Receiver to process tracing messages [default: Log]  [possible values: Log]

        --tracing-targets <TARGETS>
            Sets a custom profiling filter. Syntax is the same as for logging: <target>=<level>

        --wasm-execution <METHOD>
            Method for executing Wasm runtime code [default: Compiled]  [possible values: interpreted-i-know-
            what-i-do, compiled]
        --wasm-runtime-overrides <PATH>                          
            Specify the path where local WASM runtimes are stored.
            
            These runtimes will override on-chain runtimes when the version matches.
        --ws-max-connections <COUNT>                             
            Maximum number of WS RPC server connections

        --ws-port <PORT>                                         
            Specify WebSockets RPC server TCP port


SUBCOMMANDS:
    benchmark        Benchmark runtime pallets.
    build-spec       Build a chain specification
    check-block      Validate blocks
    export-blocks    Export blocks
    export-state     Export the state of a given block into a chain spec
    help             Prints this message or the help of the given subcommand(s)
    import-blocks    Import blocks
    key              Key management cli utilities
    purge-chain      Remove the whole chain
    revert           Revert the chain to a previous state