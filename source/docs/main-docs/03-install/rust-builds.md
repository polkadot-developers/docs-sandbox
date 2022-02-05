# Rust compiler and toolchain

Rust is a modern, type sound, and performant programming language that provides a rich feature set for building complex systems.
The language also has an active developer community and a growing ecosystem of sharable libraries called **crates**.

Rust is the core language used to build Substrate-based blockchains, so if you intend to do Substrate development, you need to be familiar with the Rust programming language, compiler, and toolchain management.

If you are just getting started with Rust, you should bookmark [The Rust Programming Language](https://doc.rust-lang.org/book/) and refer to other [Learn Rust](https://www.rust-lang.org/learn) resources on the Rust website to guide you.
However, there are a few important points to be aware of as you prepare your development environment.

The core tools in the Rust **toolchain** are the `rustc` compiler, the `cargo` build and package manager, and the `rustup` toolchain manager.
At any given point in time, there can multiple versions of Rust available. 
For example, there are release channels for stable, beta, and nightly builds.
You use the `rustup` program to manage the builds available in your environment and the versions of the toolchain programs that are used with different Rust builds.


Configure the Rust toolchain to default to the latest stable version, add nightly and the nightly wasm target:

```bash
rustup default stable
rustup update
rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
```


## Troubleshooting Substrate builds

Sometimes you can't get the [Substrate Node Template](https://github.com/substrate-developer-hub/substrate-node-template)
to compile out of the box. Here are some tips to help you work through that.

### Rust configuration check

To see what Rust toolchain you are presently using, run:

```bash
rustup show
```

This will show something like this (Ubuntu example) output:

```text
Default host: x86_64-unknown-linux-gnu
rustup home:  /home/user/.rustup

installed toolchains
--------------------

stable-x86_64-unknown-linux-gnu (default)
nightly-2020-10-06-x86_64-unknown-linux-gnu
nightly-x86_64-unknown-linux-gnu

installed targets for active toolchain
--------------------------------------

wasm32-unknown-unknown
x86_64-unknown-linux-gnu

active toolchain
----------------

stable-x86_64-unknown-linux-gnu (default)
rustc 1.50.0 (cb75ad5db 2021-02-10)
```

As you can see above, the default toolchain is stable, and the
`nightly-x86_64-unknown-linux-gnu` toolchain as well as its `wasm32-unknown-unknown` target is installed.
You also see that `nightly-2020-10-06-x86_64-unknown-linux-gnu` is installed, but is not used unless explicitly defined as illustrated in the [specify your nightly version](#specifying-nightly-version)
section.

### WebAssembly compilation

Substrate uses [WebAssembly](https://webassembly.org) (Wasm) to produce portable blockchain
runtimes. You will need to configure your Rust compiler to use
[`nightly` builds](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html) to allow you to
compile Substrate runtime code to the Wasm target.

#### Latest nightly for Substrate `master`

Developers who are building Substrate _itself_ should always use the latest bug-free versions of Rust stable and nightly. 
This is because the Substrate codebase follows the tip of Rust nightly, which means that changes in Substrate often depend on upstream changes in the Rust nightly compiler.
To ensure your Rust compiler is always up to date, you should run:

```bash
rustup update
rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
```

It may be necessary to occasionally rerun `rustup update` if a change in the upstream Substrate codebase depends on a new feature of the Rust compiler. 
When you do this, both your nightly and stable toolchains are pulled to the most recent release, and for nightly, it is generally _not_ expected to compile WASM without error (although it very often does).
Be sure to [specify your nightly version](#specifying-nightly-version) if you get WASM build errors from `rustup` and [downgrade nightly as needed](#downgrading-rust-nightly).

#### Rust nightly toolchain

If you want to guarantee that your build works on your computer as you update Rust and other
dependencies, you should use a specific Rust nightly version that is known to be
compatible with the version of Substrate they are using; this version will vary from project to
project and different projects may use different mechanisms to communicate this version to
developers. For instance, the Polkadot client specifies this information in its
[release notes](https://github.com/paritytech/polkadot/releases).

```bash
# Specify the specific nightly toolchain in the date below:
rustup install nightly-<yyyy-MM-dd>
```

#### Wasm toolchain

Now, configure the nightly version to work with the Wasm compilation target:

```bash
rustup target add wasm32-unknown-unknown --toolchain nightly-<yyyy-MM-dd>
```

### Specifying nightly version

Use the `WASM_BUILD_TOOLCHAIN` environment variable to specify the Rust nightly version a Substrate
project should use for Wasm compilation:

```bash
WASM_BUILD_TOOLCHAIN=nightly-<yyyy-MM-dd> cargo build --release
```

Note that this only builds _the runtime_ with the specified nightly. The rest of project will be compiled with **your default toolchain**, i.e. the latest installed stable toolchain.
### Downgrading Rust nightly

If your computer is configured to use the latest Rust nightly and you would like to downgrade to a
specific nightly version, follow these steps:

```sh
rustup uninstall nightly
rustup install nightly-<yyyy-MM-dd>
rustup target add wasm32-unknown-unknown --toolchain nightly-<yyyy-MM-dd>
```