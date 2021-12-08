Section: Build
Sub-section: Build process
Type: reference + conceptual
Index: ?

This article describes what happens under the hood when building a Substrate node.
One fundamental design decision in Substrate is compiling the runtime to Wasm.
This section will also go over how the runtime Wasm binary is created during the build cycle.

Relevant context:

- Substrate runtimes exist as both native and Wasm.
- Runtimes must be compressed Wasm binaries for on-chain upgrade and parachain consensus capabilities.
- Native runtime builds can be useful in some cases.

Read about the anatomy of a full Substrate node [here](/todo) in order to understand how the client interacts with the runtime. 

Note: the use of the term "Substrate node" is the same as saying "Substrate client" or "Substrate full node".

## Native and Wasm runtimes

All Substrate runtimes compile to both native ("`std`") and Wasm ("`no_std`").
This implies that the native runtime is baked into the node, while the Wasm runtime is stored on-chain as a Wasm blob. 

Having a native runtime is especially useful for development and testing environments.
On the flip side, the Wasm runtime provides the possibility of on-chain upgradability which native runtimes alone cannot. 

The design decision of having two runtime compilation formats was centred around optimizing for performance for node calls to the runtime and vice-versa.
It also gives developers the flexibility to implement different types of blockchains without any restrictions. 

A few things to consider when comparing the two. 
On the one hand:

- For development and testing, native runtimes have better debugging support, while Wasm runtimes are more difficult to debug.
- As of the time of writing, native execution has slightly better performance than Wasm execution.
- Native is better for embedded or slower hardware

On the other hand:

- Wasm only runtimes would limit the design space for developers who aren't looking for on-chain forkless upgradability (e.g. UTXO and Bitcoin-like PoW models).
- In production, native block execution doesn't improve performance: block authoring is done using the Wasm runtime.
- In production, weights are computed for Wasm execution, not Native execution. 

The native runtime is used as a regular Rust package dependency to the Substrate client.
You can see this when running `cargo tree -i node-template-runtime` from a standard Substrate node template:

```bash
# Displays all packages that depend on the node-template-runtime package.
node-template-runtime devhub/latest (...)
└── node-template devhub/latest (...)
```

The Wasm runtime is built in the initial phases of the build cycle and embedded into the client as part of the build process.
More details on this in the [next section](#build-process).

There are ongoing discussions about removing the native runtime altogether. 
Refer to this open [issue](https://github.com/paritytech/substrate/issues/7288) for more details.

### Build process

So what happens when `cargo build --release` is executed in the directory of a Substrate node?

At a very high level, this command builds the project it is being called in [with optimized artifacts](https://doc.rust-lang.org/cargo/commands/cargo-build.html#compilation-options).
These artifacts then result in the final executable program that launches a chain with the following command:
`./target/release/node-tempalte --dev`.

During the build process, the runtime Wasm binary goes through 3 different stages.
Each stage depicts the various steps in the build process.

#### Diagram [linear from A to C]:

**A. Create the initial runtime Wasm binary**

- Cargo builds the entire graph of dependencies in all project TOML files.
- Build.rs in the runtime folder executes and compiles the runtime into a Wasm binary, creating `wasm_binary.rs`.
- The build script uses the `substrate-wasm-builder` crate.
- This builds the intial wasm (largest size) which is [included in the source code](https://github.com/paritytech/substrate/blob/0e6cc5668d9ee8d852a3aa3f85a2ab5fcb4c75a1/bin/node-template/runtime/src/lib.rs#L7).

**B. Post-processing**

- Then `substrate-wasm-builder` takes the runtime code and the wasm linker invokes the optimizations to create a Compact wasm binary (target=wasm32-unknown-unknown).
- Cleans up and optimizes some instructions sequences, removes any unnecessary sections, such as the ones for debugging, using a tool called wasm-opt.

**C. Compression**

- All users should use this Wasm binary. 
    This is just a blob after applying a zstd lossless compression algorithm. 

See the sizes of each Wasm binary in Polkadot:

```bash
.rw-r--r-- 1.2M pep  1 Dec 16:13 │  ├── polkadot_runtime.compact.compressed.wasm
.rw-r--r-- 5.1M pep  1 Dec 16:13 │  ├── polkadot_runtime.compact.wasm
.rwxr-xr-x 5.5M pep  1 Dec 16:13 │  └── polkadot_runtime.wasm
```

It is important to always use the compressed version especially when maintaining a [chain in producation](./todo).
There is no need for using anything else.


<!-- WIP / notes to self:

- Wasm in storage / :code is part of the execution proof -> link to some storage article?
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
