Section: Build
Type: reference 
Index: 2

_The goal of this article is to explain how Substrate uses Rust to achieve what it provides._
_In doing so, explain how Substrate uses no_std and why, and what it has to do with compiling to Wasm_.
## Rust basics 

As a modern programming language, Rust provides a high degree of performance, type safety and memory efficiency.
Among other characteristics, it's a language which gives Substrate a powerful technological edge that other languages don't offer.

The Rust compiler helps developers be confident in the code they write, making it harder to write code with memory or concurency bugs.
This is mostly owed to its type system which solves such issues at compile time.

Useful context:

- [Rust book](https://doc.rust-lang.org/book/)

- [How rustup works](https://rust-lang.github.io/rustup/concepts/index.html)

- [Why Rust?](https://www.parity.io/blog/why-rust) 

### Cargo and crates.io

[**Cargo**](https://doc.rust-lang.org/cargo/guide/why-cargo-exists.html) is Rust's package management tool, also containing CLI tools for running tests, building documentation, benchmarks and more. 

Some common patterns for using `cargo` when developing with Substrate include:

- Generating source code documentation using `cargo doc` for any pallet or runtime.
- Running unit tests using `cargo test` for any runtime logic. 
- Updating and adding project dependencies using `cargo update` and `cargo add`.
- Using `cargo tree` for resolving dependency issues.
- Using `cargo remote` to use a faster build machine.

[**Crates.io**](https://crates.io/) is Rust's community managed package registry. 
Any Rust developer can publish their crates there for others to use in their projects. 
This is useful to make Substrate components accessible to developers and for developers to easily reuse existing modules in their projects.

### Programming paradigms

[**Types, traits and generics**](https://doc.rust-lang.org/book/ch10-00-generics.html). Rust has a sophisticated [trait system](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html) that helps developers make use of Substrate's many layers of abstraction.

[**Metaprogramming**](https://doc.rust-lang.org/book/ch19-06-macros.html). 
Substrate uses "metaprogramming" in how macros are used throughout its libraries.
This allows developers building with Substrate to write code that writes code, avoiding the need to write duplicate code.
For example, [FRAME uses macros](https://docs.substrate.io/v3/runtime/macros/) to alleviate the need to write the heavy lifting code that is required for a pallet to integrate into a runtime. 
Similarly, [ink!](https://paritytech.github.io/ink-docs/) uses macros to handle common type creations and functions.

### Webassembly 

[**Webassembly**](https://webassembly.org/). Rust compiles to executable Wasm (Webassembly) byte code, enabling Substrate runtimes to also compile to Wasm. 
Beyond being a powerful "next-generation web" technology, for Substrate, having Wasm at the core of its design means a few very specific things:

- **Upgradability**. First and foremost, Wasm is a key piece of technology for [forkless runtime upgrades](https://docs.substrate.io/v3/runtime/upgrades/).
- **Portability**. Relay chain validators use Wasm to execute any new parachain block to verify that they can actually execute it and get the same results.
- **Smart contract compatibility**. Any Smart Contract that compiles to Wasm can be executed by a compatible Substrate node. See a [list of key advantages of having Wasm smart contracts](https://paritytech.github.io/ink-docs/why-webassembly-for-smart-contracts).
- **Light-client ready**. Wasm is also a key piece in how all Substrate chains are [light-client ready out of the box](https://paritytech.github.io/substrate-connect/#wasm-light-clients). 


### Best practices

[_Note: rough sketch for now.. to move to its own dedicated "Best Practices" section imo. TBD_]

- using `saturated_add` or `checked_add` for arithmetic operations.
- error checking.
- writing tests.
Have a favorite you want to share? [Let us know](https://github.com/substrate-developer-hub/substrate-docs/issues/558)!
 
## Build environments

Rust is an embedded programming language.
This means it is designed for writing programs that don't need to rely on the standards of existing operating systems to run.
There are two classes of embedded programming environemnts: hosted environments and bare metal environments.

Hosted environments assume basic system integration primitives such as a file and memory management system (e.g. [POSIX](https://en.wikipedia.org/wiki/POSIX)) and rely on the [Rust standard library](https://doc.rust-lang.org/std/#the-rust-standard-library).
In bare metal environments, the compiled program makes no assumption about its target environment. 
This requires exclusively using the [Rust core library](https://doc.rust-lang.org/core/) for such programs and telling the compiler to ignore the standard library entirely.

For Substrate, having a bare metal environment option is a major contribution to enabling platform agnostic runtimes. 

### Compiling to `no_std` vs. `std`

Since a Substrate runtime is designed to be platform-agnostic, all runtime specific code is required to build with `no_std`.
This is done by including a default crate feature, `std` and using the [`cfg_attr` and `cfg` attributes](http://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/conditional-compilation.html) as a switch to disable `std` in favor of `no_std` where needed. 

Notice that in the Substrate node template, any file that's part of the node's runtime logic (such as `runtime/src/lib.rs` and `pallets/template/src/lib.rs`) will include:

```rust
#![cfg_attr(not(feature = "std"), no_std)]
```

This means that the code that follows this line will be treated as `no_std` except for code that is identified as _feature = "std"_.
This prevents the `std` crate from being automatically added into scope.
For code that requires the `std` crate, we simply trigger the conditional switch: 

```rust
// in runtime/src/lib.rs
#[cfg(feature = "std")]
use sp_version::NativeVersion;
```

### Wasm target

<!-- List of `no_std` known issues: https://github.com/rust-embedded/wg/issues/64 -->

By design, a Substrate node is [cross-compiled](#build-process) to embed the Wasm runtime in the client at genesis. 
To achieve this we need to specifiy a Wasm target for the compiler.
A [target](https://rust-lang.github.io/rustup/concepts/index.html) is simply information for the compiler to know what platform the code should be generated for.

Rust can support a multitude of [target platforms](https://doc.rust-lang.org/nightly/rustc/platform-support.html). 
Each target is identified as a triple which informs the compiler what kind of output a program expects.
A target triple takes the form of:

`<architecture-type>-<vendor>-<os-type>`

When [setting up your Rust environment](#installation), the compiler will default to using the host toolchain's platform as the target.
For example:

`x86_64-unknown-linux-gnu`

In order to compile to Wasm, you need to add the [`wasm32-unknown-unknown` target](https://doc.rust-lang.org/nightly/nightly-rustc/rustc_target/spec/wasm32_unknown_unknown/index.html).
This triple translates to: "_compile to a Wasm target of 32-bit address space and make no assumptions about the host and target environments_".
The result is that it can run on any type of 32-bit CPU.

[Other Wasm targets](https://docs.wasmtime.dev/wasm-rust.html) for Rust do exist. 
However, the "unknown" parts of this Wasm target enforces the notion of making zero assumptions about the target environment, which is a key design decision in Substrate.

It is also worth mentioning that there is `std` support for Substrate's Wasm target. 
But this is not something that Wasm runtimes in Substrate support as it could open up unwanted errors.
In addition, the `wasm32-unknown-unknown` target architecture and `no_std` have the same fundamental assumptions, making `no_std` a natural fit.
Rust `std` features are generally not compatible with the intended constraints of a Wasm runtime.
For example, developers who attempt operations that are not allowed in the runtime, such as printing some text using `std`, could cause potential denial of service attacks.

In general, relying only the `no_std` implementation of `wasm32-unknown-unknown` ensures:

- A Substrate runtime is deterministic.
- A Substrate runtime is platform agnostic.
- A Substrate runtime is safe from unhandled errors.

### Toolchains

Wasm runtime compilation uses [Wasm builder](https://docs.substrate.io/rustdocs/latest/substrate_wasm_builder/index.html) which requires having a nightly toolchain installed. 
This is because the `wasm32-unknown-unknown` relies on [experimental features of Rust](https://doc.rust-lang.org/unstable-book/the-unstable-book.html).
Over time, features will likely be promoted to stable.
Subscribe to [this tracking issue](https://github.com/paritytech/substrate/issues/1252) for updates.
Read more about the [build process](./build-process) to understand how a Substrate node is cross-compiled.

## Resources

- [**Cargo and crates.io**](https://doc.rust-lang.org/book/ch14-00-more-about-cargo.html)

- [**Rust blog post**](https://thenewstack.io/rust-by-the-numbers-the-rust-programming-language-in-2021/)

- [**Why Rust for smart contracts?**](https://paritytech.github.io/ink-docs/why-rust-for-smart-contracts)