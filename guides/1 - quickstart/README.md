# 1 -Quick Start

This guide provides the fastest way to get Skribe up and running using `kup`, a tool for installing and managing K framework based projects.

## Installation

### Step 1: Install kup

Install kup using the official installer:

```bash
bash <(curl https://kframework.org/install)
```

### Step 2: Verify kup Installation

Test that kup is working correctly:

```bash
kup list
```

This command lists all available K framework projects that can be installed via kup.

### Step 3: Install Skribe

Install the latest version of Skribe:

```bash
kup install skribe
```

This installs Skribe along with all required dependencies and automatically configures the environment.

### Step 4: Verify Skribe Installation

Test that Skribe is installed correctly:

```bash
skribe --help
```

You should see the Skribe command help output listing the available `build` and `run` subcommands.

## Running Your First Test

Skribe includes example test contracts in its [GitHub repository](https://github.com/runtimeverification/skribe/). We recommend starting with the `test-hello-world` contract located at `src/tests/integration/data/contracts/test-hello-world`.

### Step 1: Navigate to a Test Contract

```bash
cd src/tests/integration/data/contracts/test-hello-world
```

### Step 2: Build the Test Contract

```bash
skribe build
```

This compiles the test contract to WebAssembly.

### Step 3: Run a Test

Run a specific test function with fuzzing:

```bash
skribe run --id testSelfNumber --max-examples 200
```

This executes the `testSelfNumber` test function with 200 randomized input examples. Skribe will display a progress bar and report the results.

**Note**: Test function IDs use camelCase format. If a Rust function named `test_self_number` it will be referenced as `testSelfNumber` when using the `--id` flag.

### Running All Tests

To run all test functions in the contract:

```bash
skribe run --max-examples 200
```

## Next Steps

Now that you have Skribe installed and running, you can:

- Learn how to write your own test contracts in the [Writing Tests section](https://github.com/runtimeverification/gitbook-skribe/blob/main/guides/3%20-%20writing-tests/README.md), as well as take a look at the [Usage section](https://github.com/runtimeverification/gitbook-skribe/blob/main/guides/4%20-%20usage/README.md) to learn more about the different options available to run tests.
- Explore the example test contracts in Skribe's repository ([GitHub link](https://github.com/runtimeverification/skribe/tree/master/src/tests/integration/data/contracts)).
- Review available cheatcodes for advanced testing scenarios in the [Cheatcodes section](https://github.com/runtimeverification/gitbook-skribe/blob/main/guides/5%20-%20cheatcodes/README.md).

## Getting Help

If you encounter any issues while following this quick start guide, you can:

- Report issues on the [Skribe GitHub repository](https://github.com/runtimeverification/skribe)
- Reach out to Runtime Verification via Discord or Twitter

