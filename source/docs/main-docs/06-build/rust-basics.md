Section: Build
Type: reference 
Index: 2

## Rust basics and best practices

Beyond the fact that Rust provides high performance, type safety and memory efficiency, it provides Substrate with a powerful technological edge that other languages don't offer.

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
This is useful to make Substrate accessible to developers and for developers to reuse existing modules in their projects.

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

Some best practices using Rust with Substrate: 

- using `saturated_add` or `checked_add` for arithmetic operations.
- error checking 

See this: https://github.com/substrate-developer-hub/substrate-docs/issues/558. 

Links:

[**Cargo and crates.io**](https://doc.rust-lang.org/book/ch14-00-more-about-cargo.html)

[**Rust blog post**](https://thenewstack.io/rust-by-the-numbers-the-rust-programming-language-in-2021/)

[**Why Rust for smart contracts?**](https://paritytech.github.io/ink-docs/why-rust-for-smart-contracts)

### Compiling to no_std vs. std

Also about std vs. no_std: https://github.com/substrate-developer-hub/substrate-docs/issues/531.

