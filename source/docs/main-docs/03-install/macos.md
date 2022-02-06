# macOS development environment

You can install Rust and set up a Substrate development environment on Apple macOS computers with either Intel or an Apple M1 processors.

## Before you begin

Before you install Rust and set up your development environment on macOS, verify that your computer meets the following basic requirements:

* Operating system version is 10.7 Lion, or later.
* Processor speed of at least 2Ghz, 3Ghz recommended.
* Memory of at least 8 GB RAM, 16 GB recommended.
* Storage of at 10 GB available space.
* Broadband Internet connection.

### Support for Apple M1

If you have a macOS computer with an Apple M1 ARM system on a chip, you must have Apple Rosetta installed to run the `protoc` tool during the Rust build process.
The Rust toolchain and the target binaries remain native.
You can install Rosetta on your macOS computer by opening the Terminal application and running the following command:

`softwareupdate --install-rosetta`

## Install Homebrew

In most cases, you should use Homebrew to install and manage packages on macOS computers.
If you don't already have Homebrew installed on your local computer, you should download and install it before continuing.

To install Homebrew:

1. Open the Terminal application.

1. Download and install Homebrew by running the following command:
    
    ```
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
    ```

1. Verify Homebrew has been successfully installed by running the following command:
    
    ```
    brew --version
    ```

    The command displays output similar to the following:

    ```
    Homebrew 3.3.1
    Homebrew/homebrew-core (git revision c6c488fbc0f; last commit 2021-10-30)
    Homebrew/homebrew-cask (git revision 66bab33b26; last commit 2021-10-30)
    ```

## Install required packages and Rust

Because the blockchain requires standard cryptography to support the generation of public/private key pairs and the validation of transaction signatures, you must also have a package that provides cryptography, such as `openssl`.

To install `openssl` and the Rust toolchain on macOS:

1. Open the Terminal application.

1. Ensure you have an updated version of Homebrew by running the following command:
    
    ```
    brew update
    ```

1. Install the `openssl` package by running the following command:
    
    ```
    brew install openssl
    ```

1. Download the `rustup` installation program and use it to install Rust by running the following command:
    
    ```bash
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    ```

1. Follow the prompts displayed to proceed with a default installation.

1. Update your current shell to include Cargo by running the following command:
    
    ```bash
    source ~/.cargo/env
    ```

1. Verify your installation by running the following command:
    
    ```bash
    rustc --version
    ```

1. Configure the Rust toolchain to default to the latest stable version by running the following commands:
    
    ```bash
    rustup default stable
    rustup update
    ```

1. Add the `nightly` release and the `nightly` WebAssembly (wasm) targets to your development environment by running the following commands:
    
    ```bash
    rustup update nightly
    rustup target add wasm32-unknown-unknown --toolchain nightly
    ```

1. Verify the configuration of your development environment by running the following command:
    
    ```bash
    rustup show
    ```

    The command displays output similar to the following:

    <pre>
    Default host: x86_64-apple-darwin
    rustup home:  /Users/lisagunn/.rustup
    
    installed toolchains
    --------------------
    
    stable-x86_64-apple-darwin (default)
    nightly-x86_64-apple-darwin
    
    active toolchain
    ----------------
    
    stable-x86_64-apple-darwin (default)
    rustc 1.58.1 (db9d1b20b 2022-01-20)
    </pre>
