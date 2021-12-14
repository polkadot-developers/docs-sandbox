# Windows development environment

Supported architecture and versions 

Installation instructions

Verifying your installation

Troubleshooting

In general, Substrate development is best supported on UNIX-based operating systems like macOS or Linux and all of the examples in Substrate [Tutorials](../tutorials) and [How-to guides](../reference/how-to-guides) illustrate how to use UNIX-compatible commands to interact with Substrate from a terminal. 

If you only have Microsoft Windows available on your local computer, there are configuration steps required to prepare your environment for Substrate development.
To develop on Windows, you have the following options:

* Use Windows Subsystem Linux (WSL) to emulate a UNIX operating environment (recommended).

* Add all of the required packages to the native Windows environment.

## Install and use Windows Subsystem Linux

To use Windows Subsystem Linux to emulate Linux:

1. Download [Windows Subsystem Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

1. Open a Command Prompt window.

1. ollow the instructions for [Ubuntu/Debian](../installation#ubuntudebian).

## Install packages for a native Windows environment

If you want to use a Windows computer to _natively_ build Substrate, you need to download and install several required packages.

To prepare Windows for Substrate development:

1. Verify that you have [Visual Studio]() installed as your integrated development environment.

1. Download [Build Tools for Visual Studio](https://aka.ms/buildtools).

1. Open a Command Prompt window and change to the Downloads directory where you downloaded the build tools for Visual Studio.

1. Install the build tools by running the following command:

    ```dos
    vs_buildtools.exe
    ```

1. Verify the **Windows 10 SDK** component is included in the installation of the Visual C++ Build Tools.

1. Restart your computer.

### Install Rust on Windows

   - Detailed instructions are provided by the
     [Rust Book](https://doc.rust-lang.org/book/ch01-01-installation.html#installing-rustup-on-windows)
     and a quick reference is available at <https://rustup.rs/> .

     - Download from: https://www.rust-lang.org/tools/install.
     - Run the installation file: `rustup-init.exe` for 32 or 64 bis as appropriate.
       - It shouldn't prompt you to install `vs_buildtools` since you did it in step 1.
     - Choose "Default Installation."
     - To get started, you need Cargo's bin directory (`%USERPROFILE%\.cargo\bin`) in your PATH
       environment variable. Future applications will automatically have the correct environment,
       but you may need to restart your current shell.

3. Run these commands in Command Prompt (`CMD`) to set up your Wasm Build Environment:

   ```bash
   rustup update nightly
   rustup update stable
   rustup target add wasm32-unknown-unknown --toolchain nightly
   ```

### Install and configure additional programs

1. Install LLVM: https://releases.llvm.org/download.html

1. Install OpenSSL with `vcpkg` using PowerShell:

   ```bash
   mkdir C:\Tools
   cd C:\Tools
   git clone https://github.com/Microsoft/vcpkg.git --depth=1
   cd vcpkg
   .\bootstrap-vcpkg.bat
   .\vcpkg.exe install openssl:x64-windows-static
   ```

1. Add OpenSSL to your System Variables using PowerShell:

   ```powershell
   $env:OPENSSL_DIR = 'C:\Tools\vcpkg\installed\x64-windows-static'
   $env:OPENSSL_STATIC = 'Yes'
   [System.Environment]::SetEnvironmentVariable('OPENSSL_DIR', $env:OPENSSL_DIR, [System.EnvironmentVariableTarget]::User)
   [System.Environment]::SetEnvironmentVariable('OPENSSL_STATIC', $env:OPENSSL_STATIC, [System.EnvironmentVariableTarget]::User)
   ```

1. Install `cmake`: https://cmake.org/download/

1. Install `make`

   - This can be done using Chocolatey. First you need to install the Chocolatey package manager: https://chocolatey.org/install
   - Once Chocolatey installed you can install make:

   ```
   choco install make
   ```