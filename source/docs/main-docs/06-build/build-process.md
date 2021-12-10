Section: Build
Sub-section: Build process
Type: reference + conceptual
Index: ?

This article describes what happens under the hood when building a Substrate node.

Relevant context:

- Substrate runtimes are compiled to [both native and Wasm](#native-and-wasm-runtimes).
- Runtimes must be compressed Wasm binaries for on-chain upgrades and parachain consensus capabilities.
- Read about the anatomy of a full Substrate node [here](/todo) in order to understand how the client interacts with the runtime. 

**Note:** the use of the term "Substrate node" is the same as saying "Substrate client" or "Substrate full node".

## Native and Wasm runtimes
_Notes: should go to fundamentals or Rust section. Or move this to architecture page. The goal of this section is to explain why Substrate is designed to have both Wasm and native runtimes, as a prelude to the build process._

All Substrate runtimes compile to both native ("`std`") and Wasm ("`no_std`") environments.
<!--This implies that the native runtime is baked into the node, while the Wasm runtime is stored on-chain as a Wasm blob.  -->

Considered an optimization to Substrate, the native runtime is especially useful for development and testing environments.
It is optional in the sense that production chains don't need to rely on native builds.
The Wasm runtime on the other hand is not optional. 
A chain's Wasm binary is embedded in the client at compile time and required for any [chain specification](./todo).
Even when choosing to skip compiling the Wasm runtime for developement purposes, Substrate will include a dummy Wasm blob otherwise the chain will panic.
Central to Substrate's desgin, it provides the possibility of on-chain upgradability and multi-chain communication.

### Why two runtimes?

There are ongoing discussions about removing the native runtime altogether. 
Refer to this open [issue](https://github.com/paritytech/substrate/issues/7288) for more details.

Here are some reasons why using a native runtime could be desired:

- For development and testing, native runtimes have better debugging support, while Wasm runtimes are more difficult to debug.
- As of the time of writing, native execution has slightly better performance than Wasm execution.
- Native is generally better for slower hardware.
- Native runtimes provide a less opiniated building framework, while Wasm only runtimes would limit the design space for developers who aren't looking for on-chain forkless upgradability (e.g. UTXO and Bitcoin-like PoW models).

However:

- Native runtimes can only be compiled once.
- In production, native block execution doesn't improve overall performance when factoring in on-chain upgrades which can only be done with Wasm runtimes.
- In production, weights are computed for Wasm execution anyway, not native execution.

## Build process

The native runtime is used as a regular Rust package dependency to the Substrate client.
You can see this when running `cargo tree -i node-template-runtime` from a standard Substrate node template:

```bash
# Displays all packages that depend on the node-template-runtime package.
node-template-runtime devhub/latest (...)
└── node-template devhub/latest (...)
```

So what happens when `cargo build --release` is executed in the directory of a Substrate node template?

At a very high level, this command builds the project it is being called in with [optimized artifacts](https://doc.rust-lang.org/cargo/commands/cargo-build.html#compilation-options).
These artifacts then result in the final executable program that enables launching a chain with the following command:
`./target/release/node-template --dev`.

During the build process, the runtime Wasm binary goes through 3 different stages, whereby each stage depicts the various steps in the build process.
The initial Wasm runtime is built in the first stage of the build cycle and embedded into the client.
Once the entire Rust program is compiled, a lighter weight Wasm binary is stored on-chain from the genesis configuration of the chain at the [`:code`](https://docs.substrate.io/rustdocs/latest/sp_storage/well_known_keys/constant.CODE.html) storage key. 

[ note: turn below into diagram wip]

┌───────┐      ┌───────┐
│       │      │       │
│   A   ├─────►│   B   │
│       │      │       │
└───────┘      └───────┘

**A. Create the initial runtime Wasm binary**

- Cargo builds the entire graph of dependencies in all project TOML files.
- Build.rs for runtime uses the `substrate-wasm-builder` crate.
- It executes and compiles the runtime into a Wasm binary, creating `wasm_binary.rs`, i.e. the intial wasm (largest size). 
- The runtime crate requires this Wasm to [embed it](https://github.com/paritytech/substrate/blob/0e6cc5668d9ee8d852a3aa3f85a2ab5fcb4c75a1/bin/node-template/runtime/src/lib.rs#L7) into its compilation result


**B. Post-processing**

- Then the `substrate-wasm-builder` wasm linker invokes the optimizations to create a Compact wasm binary.
- It optimizes some instruction sequences, removes any unnecessary sections, such as the ones for debugging, using a tool called wasm-opt.
- The runtime crate is a dependency of the node.

**C. Compression**

- A [zstd lossless compression](https://en.wikipedia.org/wiki/Zstandard) algorithm is applied to minimize the size of the final Wasm binary. 
- All users should use this Wasm binary. 

**D. Result**

- The final executable binary is `node-template`.
- The `./target/release/node-template --dev` command initializes a new chain, i.e. generates a new chainspec.
- The Wasm runtime is put as an item in storage (with the magic key named “:code”)
- Chain spec has the genesis state and includes the Wasm binary, which was fetched from the node-runtime crate.


See the sizes of each Wasm binary in Polkadot:

```bash
.rw-r--r-- 1.2M pep  1 Dec 16:13 │  ├── polkadot_runtime.compact.compressed.wasm
.rw-r--r-- 5.1M pep  1 Dec 16:13 │  ├── polkadot_runtime.compact.wasm
.rwxr-xr-x 5.5M pep  1 Dec 16:13 │  └── polkadot_runtime.wasm
```

It is important to always use the compressed version especially for maintaining a [chain in production](./todo).
There is no need for using any other of the Wasm artifacts.

See the [README for the Wasm builder](https://github.com/paritytech/substrate/blob/master/utils/wasm-builder/README.md).

## Build options

It can make sense to compile the Wasm binary only, if for example you are just trying to provide an upgraded Wasm to perform a forkless upgrade. 
Usually when performing a runtime upgrade, you want to provide both a native and Wasm binary.
Doing only one of those halves usually is an incomplete release process.

In any case, when starting a new chain the initial Wasm binary is required. 
In production the Wasm runtime comes from the chain specification of a chain.
However, when starting a chain in developer mode at block 0, it uses the embedded Wasm from the native runtime.

There are several ways to configure a chain to meet the requirements of your needs:

- `SKIP_WASM_BUILD` - Skips building any Wasm binary. This is useful when only native should be recompiled.
                      If this is the first run and there doesn't exist a Wasm binary, this will set both variables to `None`.

- Use [this script](https://github.com/paritytech/substrate/blob/master/.maintain/build-only-wasm.sh
build-only-wasm.sh node-runtime ./output.wasm
) to build the no_std Wasm binary only. 

- Use the [wasm-runtime-overrides] CLI flag to load the Wasm from the filesystem.

<!-- WIP / notes to self:

- Executor and runtime interface parts of substrate client/runtime: link to node architecture?
- State machine & executor: diagram potential
- Wasmtime for execution
- For parachain validation, the runtime code is part of the validation blob that is stored on the relay chain. So, we don't need :code to be part of the proof. (see: https://github.com/paritytech/substrate/issues/5047)
- Why sr tool is important? Link to tooling

## sr tool

- allows building WASM runtimes in a deterministic way, producing two identical wasm runtimes
- you can use GH actions to verify determinism

## Chain specification

## CLI 

- ./target/release/parachain-collator export-genesis-wasm --chain parachain-raw.json > para-wasm
- --chain parachain-raw.json \  -->
