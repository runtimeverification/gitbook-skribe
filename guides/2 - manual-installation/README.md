# 2 - Manual Installation

This guide covers building and installing Skribe from source. For prerequisites, refer to the official documentation of each tool.

## Prerequisites

Before building Skribe, ensure you have the following installed and configured:

- **K framework**: Install from [GitHub releases](https://github.com/runtimeverification/k/releases/latest)
- **Python**: Version 3.10 or higher. See [Python documentation](https://www.python.org/downloads/)
- **pip**: Version 20.0.2 or higher. Usually included with Python
- **uv**: Install from [uv documentation](https://docs.astral.sh/uv/)
- **Rust**: With `wasm32-unknown-unknown` target. See [Rust installation guide](https://www.rust-lang.org/tools/install)
- **Cargo Stylus**: Install via `cargo install cargo-stylus`. See [Cargo Stylus repository](https://github.com/OffchainLabs/cargo-stylus)
- **Solidity Compiler**: Version 0.8.23 or compatible. See [Solidity documentation](https://docs.soliditylang.org/en/latest/installing-solidity.html)
- **Xcode Command Line Tools**: Required on macOS. Install via `xcode-select --install`

## Building Skribe

### Step 1: Clone the Repository

```bash
git clone <repository-url>
cd skribe
```

### Step 2: Build the Distribution Package

Build the Python wheel package:

```bash
make build
```

This creates distribution files in the `dist/` directory.

### Step 3: Build Semantics Definitions

Build the Stylus semantics definitions required for execution:

```bash
make kdist-build
```

This step builds the `stylus-semantics.llvm` definition and may take 10 to 20 minutes on first run as it compiles C++ dependencies.

**Note for Apple Silicon users**: The Makefile automatically sets `SDKROOT` and `APPLE_SILICON=true`. If you encounter build errors, you can manually set these:

```bash
SDKROOT=$(xcrun --show-sdk-path) APPLE_SILICON=true make kdist-build
```

## Installing Skribe

### Step 4: Install the Package

Install Skribe using Python's pip module to ensure installation to the correct Python environment:

```bash
python3 -m pip install dist/*.whl
```

This installs Skribe along with its dependencies including `kontrol`, `pykwasm`, and `kevm-pyk`.

### Step 5: Verify Installation

Test that Skribe is installed correctly:

```bash
skribe --help
```

You should see the Skribe command help output with `build` and `run` subcommands.

### Step 5 (**if necessary**): Troubleshooting Skribe's automatic configuration

If encountering errors related to a `Target undefined or not built`, more specifically the `kdist` `stylus-semantics.llvm`, set the `KDIST_DIR` environment variable to point to where the semantics were built. The build directory is typically located in `~/.cache/kdist-*/`.

To do so, add this to your shell configuration file (`~/.zshrc` for zsh or `~/.bashrc` for bash):

```bash
export KDIST_DIR=$(ls -td ~/.cache/kdist-* 2>/dev/null | head -1)
```

Then reload your shell:

```bash
source ~/.zshrc  # or source ~/.bashrc
```

**Manual configuration**

Find the kdist directory:

```bash
ls -td ~/.cache/kdist-* | head -1
```

Set the environment variable in your current session or add to your shell profile:

```bash
export KDIST_DIR=/path/to/kdist-XXXXX
```

## Optional: Installing KEVM

If you need KEVM semantics for EVM testing, or if you are a Kontrol user, install it with a lower priority to coexist with kontrol and skribe:

```bash
nix profile install "github:runtimeverification/evm-semantics/769d7ed1db93d20db23bac156ce03f80690ccb9a?narHash=sha256-ph4iPU9F6cL7gigjw1kUjyGR/c4Zd9ggvx1XAjQ7lCA%3D#packages.aarch64-darwin.kevm" --priority 6 --extra-experimental-features "nix-command flakes"
```

Verify KEVM installation:

```bash
kevm version
```

## Quick Reference

Complete manual build/installation sequence:

```bash
# Build Skribe
make build
make kdist-build

# Install package
python3 -m pip install dist/*.whl

# If necessary, configure environment (add to shell profile)
export KDIST_DIR=$(ls -td ~/.cache/kdist-* 2>/dev/null | head -1)

# Verify
skribe --help
```
