# 4 - Usage

This guide covers how to use Skribe's command-line interface to build and run tests.

## Command Overview

Skribe provides two main commands:

- `skribe build`: Compiles test contracts to WebAssembly;
- `skribe run`: Executes fuzz tests on compiled contracts.

Both commands support the `--help` flag to display detailed usage information.

## Building Test Contracts

The `build` command compiles a test contract located in a specified directory.

### Basic Usage

```bash
skribe build
```

When run without arguments, Skribe builds the contract in the current working directory.

### Options

- `--directory DIRECTORY`, `-C DIRECTORY`: Specify the test contract directory. Defaults to the current working directory.

### Example

```bash
skribe build --directory path/to/test/contract
```

## Running Tests

The `run` command executes fuzz tests on a compiled test contract. It automatically discovers test functions and executes them with randomized inputs.

### Basic Usage

```bash
skribe run
```

This runs all discovered test functions in the current directory with the default number of fuzzing examples.

### Options

- `--directory DIRECTORY`, `-C DIRECTORY`: Specify the test contract directory. Defaults to the current working directory;
- `--id ID`: Run a specific test function by name. If not specified, all test functions are executed;
- `--max-examples MAX_EXAMPLES`: Maximum number of fuzzing inputs to generate. Default is 100.

### Test Execution Flow

When you run `skribe run`, it performs the following steps:

1. **Create contracts**: Reads configuration files and deploys any required contract dependencies;
2. **Initialize the test contract**: Deploys the test contract and calls its initialization function if present;
3. **Discover test functions**: Scans for functions with the `test_` prefix;
4. **Execute fuzz tests**: Runs each test function with randomized inputs up to the specified maximum;
5. **Report results**: Displays progress bars and reports any test failures.

### Examples

Run all tests with default settings:

```bash
skribe run
```

Run a specific test function:

```bash
skribe run --id testSelfNumber
```

Run with custom fuzzing parameters:

```bash
skribe run --id testCallSetGetNumber --max-examples 500
```

Run tests in a different directory:

```bash
skribe run --directory path/to/contract --max-examples 200
```

### Test Function Naming

Test function IDs use camelCase format. When referencing a Rust function named `test_self_number` in your code, use `testSelfNumber` with the `--id` flag.

### Understanding Output

During execution, Skribe displays progress bars for each test function. A test passes when it completes without panicking. A test fails if it panics or returns an error, at which point Skribe reports the failing input and execution details.

## Common Workflows

### Complete Test Cycle

```bash
# Build the contract
skribe build --directory ./my-contract

# Run all tests
skribe run --directory ./my-contract --max-examples 200

# Run a specific failing test with more examples
skribe run --directory ./my-contract --id testSpecificFunction --max-examples 1000
```

### Quick Iteration

When working in the contract directory, you can omit the directory flag:

```bash
cd path/to/contract
skribe build
skribe run --max-examples 100
```

### Testing Specific Scenarios

To focus on a particular test function during development:

```bash
skribe run --id testEdgeCase --max-examples 50
```

## Tips

- Build contracts before running tests. The `run` command requires a compiled contract;
- Use `--max-examples` to control test execution time. Higher values provide more thorough testing but take longer;
- Test function discovery is automatic. Any function starting with `test_` will be found and executed;
- Progress bars show real-time execution status for each test function;
- Test failures include the specific input that caused the failure, making debugging easier.

