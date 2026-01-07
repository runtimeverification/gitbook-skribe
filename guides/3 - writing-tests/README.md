# 3 - Writing Tests

This guide explains how to write test contracts for Skribe. Skribe enables property testing and fuzzing for Stylus smart contracts, allowing you to discover edge cases and validate contract behavior through automated test execution.

Skribe supports writing tests in both Solidity and Rust (Stylus), and importantly, enables cross-testing where tests written in one language can interact with contracts written in the other. This flexibility allows you to test your contracts using the tools and patterns you're most comfortable with, while still being able to verify interactions between different contract types.

## Cross Testing Support

Skribe's cross-testing capabilities enable two primary scenarios:

- **Solidity tests interacting with Stylus contracts**: Write Foundry-style tests in Solidity that deploy and interact with Stylus contracts compiled to WebAssembly;
- **Stylus tests interacting with Solidity contracts**: Write test contracts in Rust that deploy and interact with traditional Solidity contracts.

This cross-testing support is particularly valuable when:
- You have a mixed codebase with both Solidity and Stylus contracts that need to interact;
- You want to test integration points between different contract types;
- You prefer writing tests in one language but need to verify contracts written in another;
- You're migrating from Solidity to Stylus and need to ensure compatibility.

Tests are written using the native conventions of each ecosystem. Skribe does not introduce a foreign testing model. Solidity developers use standard Foundry patterns and cheatcodes, while Stylus developers write tests as normal Stylus contracts with additional testing features implemented as cheatcodes, analogous to Foundry's cheatcode system.

## Solidity Tests

You can write tests using standard Foundry conventions for testing your Solidity contracts. All major Foundry cheatcodes are fully supported, so if you're familiar with Foundry testing, you can apply the same patterns and techniques when using Skribe. To get a better perspective of supported cheatcodes and how to refer to them, see the [Cheatcodes section](./../5%20-%20cheatcodes/README.md) of this document.

### Test Structure

Solidity tests follow the usual Foundry style with familiar conventions:

- **`setUp()` function**: Called once before each test function to initialize test state;
- **`test_*` functions**: Individual test cases that are automatically discovered and executed;
- **Standard deployment patterns**: Use the `new` keyword to deploy contracts, just like in Foundry;
- **Cheatcode access**: Access Foundry cheatcodes via the `vm` object, including `vm.assume()`, `vm.prank()`, `vm.deal()`, and more.

### Basic Example

Here's a simple example demonstrating a Foundry-style test:

```solidity
import "forge-std/Test.sol";

contract Counter {
    uint256 public number;

    function setNumber(uint256 newNumber) public {
        number = newNumber;
    }

    function increment() public {
        number++;
    }
}

contract MyTest is Test {
    Counter counter;

    function setUp() public {
        counter = new Counter();
    }

    function test_increment(uint256 x) public {
        counter.setNumber(x);
        counter.increment();
        assertEq(counter.number(), x + 1);
    }
}
```

This example shows the standard Foundry pattern: inherit from `Test`, use `setUp()` for initialization, and write test functions with the `test_` prefix. The test function takes a parameter `x`, which Skribe will fuzz with randomized values.

### Deploying Solidity Contracts

Deploy Solidity contracts using the standard `new` keyword, exactly as you would in Foundry:

```solidity
Counter counter = new Counter();
```

This creates a new instance of the contract and stores its address. You can also pass constructor arguments:

```solidity
MyContract contract = new MyContract(arg1, arg2);
```

### Deploying Stylus Contracts from Solidity

To deploy a Stylus contract from a Solidity test, you need to read the WASM bytecode file and wrap it in EVM constructor code that includes the Stylus-specific prelude and version bytes. This process is necessary because Stylus contracts are compiled to WebAssembly, but they must be deployed through the EVM with a specific format.

**Step 1: Read the WASM file**

Use Foundry's `vm.readFileBinary()` cheatcode to read the WASM bytecode:

```solidity
bytes memory bytecode = vm.readFileBinary("wasm/stylus_hello_world.wasm");
```

The path is relative to your project root. Make sure the WASM file exists at the specified location.

**Step 2: Wrap the bytecode in EVM constructor code**

The constructor code must include the Stylus prelude and version bytes. This wrapper code tells the EVM how to properly deploy the Stylus contract:

```solidity
bytes memory initCode = abi.encodePacked(
    hex"7f",              // PUSH32
    bytecode.length + 4,  // length(prefix + bytecode)
    hex"80",              // DUP1
    hex"60",              // PUSH1
    bytes1(uint8(42 + 1)), // prelude + version
    hex"60", hex"00",     // PUSH1 0x00
    hex"39",              // CODECOPY
    hex"60", hex"00",     // PUSH1 0x00
    hex"f3",              // RETURN
    hex"00",              // version
    hex"eff000",          // Stylus discriminant
    hex"00",              // compression level
    bytecode
);
```

This assembly code sets up the proper deployment format that Stylus contracts require. The specific byte sequences identify the contract as a Stylus contract and ensure it's deployed correctly.

**Step 3: Deploy using the `create` instruction**

Use inline assembly to deploy the contract:

```solidity
address newContractAddress;
assembly {
    newContractAddress := create(0, add(initCode, 0x20), mload(initCode))
}
```

The `create` instruction takes three arguments: the value to send (0 in this case), the memory offset where the init code starts, and the length of the init code. After deployment, `newContractAddress` contains the address of the newly deployed Stylus contract.

**Complete Example**

In [Skribe's GitHub repository](https://github.com/runtimeverification/skribe), see the example in `src/tests/integration/data/contracts/test-stylus-from-foundry/test/StylusHelloWorld.t.sol` for a complete implementation that demonstrates deploying a Stylus contract and interacting with it from a Solidity test (link).

## Stylus Tests

Tests are written as Stylus contracts using the `skribe-rs` library, which provides cheatcode interfaces and deployment helpers. If you're familiar with writing Stylus contracts, writing tests follows the same patterns, with the addition of testing specific functionality through cheatcodes.

### Test Contract Structure

Stylus test contracts mirror Foundry's structure, providing a familiar testing experience:

- **`set_up()` function**: Called once before each test function to initialize test state (note the snake_case naming, which is Rust convention);
- **Public `test_*` functions**: Individual test cases that are automatically discovered and executed by Skribe;
- **Cheatcode access**: Access testing cheatcodes via `skribe::cheat()`, similar to Foundry's `vm` object.

### Basic Example Structure

Here's a minimal example showing the structure of a Stylus test contract:

```rust
use stylus_sdk::prelude::*;
use skribe::cheat;

sol_storage! {
    #[entrypoint]
    pub struct TestContract {
        uint256 value;
    }
}

#[public]
impl TestContract {
    pub fn set_up(&mut self) {
        self.value.set(U256::from(100));
    }

    pub fn test_example(&mut self, x: U256) {
        assert_eq!(x, x);
    }
}
```

The `sol_storage!` macro defines the contract's storage layout, and the `#[entrypoint]` attribute marks it as the main contract entry point. Test functions are public methods in the `impl` block, prefixed with `test_`.

### Using Cheatcodes

Access cheatcodes through the `skribe::cheat()` function, which returns an interface to all available cheatcode operations. Cheatcodes allow you to manipulate blockchain state, read files, control execution flow, and more.

**Common cheatcodes include:**

- **`assume()`**: Discard fuzz inputs that don't meet a condition, generating new ones instead;
- **`deal()`**: Set an address's balance;
- **`prank()` / `startPrank()` / `stopPrank()`**: Manipulate `msg.sender` for subsequent calls;
- **`warp()`**: Set `block.timestamp`;
- **`roll()`**: Set `block.number`;
- **`store()` / `load()`**: Directly manipulate storage slots;
- **`readFile()` / `readFileBinary()`**: Read files from the filesystem;
- **`expectRevert()`**: Expect the next call to revert.

For more information on the cheatcodes available, see the [Cheatcodes](../5%20-%20cheatcodes/README.md) guide.

**Example using `assume()`:**

```rust
use skribe::cheat;

pub fn test_with_assume(&mut self, x: U256) {
    // Only test with values less than MAX
    cheat().assume(&mut *self, x < U256::MAX).unwrap();
    
    // Test logic that assumes x < U256::MAX
    let result = x.checked_add(U256::from(1));
    assert!(result.is_some());
}
```

The `assume()` cheatcode is particularly useful for property testing, as it allows you to filter out invalid inputs and focus the fuzzer on interesting cases.

**Example using file reading:**

```rust
pub fn test_with_file(&mut self) {
    let content = cheat()
        .read_file(&mut *self, "data/config.json".to_string())
        .unwrap();
    // Use the file content in your test
}
```

### Deploying Solidity Contracts from Rust

To deploy a Solidity contract from a Stylus test, you need to compile the Solidity contract to binary format, load it, and deploy it using the VM interface.

**Step 1: Compile the Solidity contract**

First, compile your Solidity contract using Foundry with the `--extra-output-files bin` flag:

```bash
forge build --extra-output-files bin
```

This creates a `.bin` file in the `out/` directory (or `bin/` if configured) containing the hex encoded bytecode of your compiled contract. The file will be named after your contract, for example `Counter.bin`.

**Step 2: Load the bytecode file**

Use the `read_file()` cheatcode to load the hex encoded bytecode:

```rust
let hex_bytecode = cheat()
    .read_file(&mut *self, "bin/Counter.bin".to_string())
    .unwrap();
```

The path is relative to your project root. The file contains a hex string representation of the contract's bytecode.

**Step 3: Decode the hex string**

Convert the hex string to bytes using a hex decoding library:

```rust
let bytecode = hex::decode(hex_bytecode).unwrap();
```

Make sure you have the `hex` crate in your `Cargo.toml` dependencies if it's not already included.

**Step 4: Deploy using the VM interface**

Deploy the contract using `self.vm().deploy()`:

```rust
let counter_address = unsafe {
    self.vm()
        .deploy(&bytecode, U256::ZERO, Option::None)
        .unwrap()
};
```

The `deploy()` method takes the bytecode, the value to send (U256::ZERO for no value), and an optional salt for CREATE2 deployments. The method returns the address of the deployed contract.

**Complete Example**

In [Skribe's GitHub repository](https://github.com/runtimeverification/skribe), see the example in `src/tests/integration/data/contracts/test-foundry-from-stylus/src/lib.rs` for a complete implementation that demonstrates deploying a Solidity contract and interacting with it from a Stylus test.

### Deploying Stylus Contracts from Rust

To deploy a Stylus contract from another Stylus test, you need to read the WASM file and wrap it with the proper initialization code using Skribe's helper function.

**Step 1: Read the WASM file**

Read the WASM file as binary using `read_file_binary()`:

```rust
let bytecode = cheat()
    .read_file_binary(&mut *self, "path/to/contract.wasm".to_string())
    .unwrap();
```

The path is relative to your project root. This reads the raw WebAssembly bytecode.

**Step 2: Wrap the bytecode**

Use Skribe's `build_init_code()` helper function to wrap the bytecode with the necessary Stylus deployment prefix:

```rust
use skribe::build_init_code;

let init_code = build_init_code(&bytecode);
```

This function automatically adds the required Stylus prelude, version bytes, and constructor wrapper, so you don't need to manually construct the assembly code.

**Step 3: Deploy using the VM interface**

Deploy the contract using `self.vm().deploy()`:

```rust
let contract_address = unsafe {
    self.vm()
        .deploy(&init_code, U256::ZERO, Option::None)
        .unwrap()
};
```

The deployment process is the same as deploying Solidity contracts, but you use the wrapped init code instead of raw bytecode.

**Complete Example**

In [Skribe's GitHub repository](https://github.com/runtimeverification/skribe), see the example in `src/tests/integration/data/contracts/test-hello-world/src/lib.rs` for a complete implementation that demonstrates deploying a Stylus contract from another Stylus test.

## Test Function Conventions

Understanding Skribe's test function conventions is important for writing tests that are properly discovered and executed.

### Function Naming

Test functions must follow these naming rules:

- **Must start with `test_` prefix**: Skribe automatically discovers functions that begin with `test_`;
- **Should be public**: Test functions must be accessible, so they should be marked as `pub` in Rust;
- **Return unit type `()`**: Test functions should not return values; they indicate success by not panicking.

**Examples:**

```rust
// Valid test function
pub fn test_my_feature(&mut self) {
    // test logic
}

// Valid test function with parameters
pub fn test_with_params(&mut self, x: U256, y: U256) {
    // test logic
}

// Invalid: missing test_ prefix (won't be discovered)
pub fn my_test(&mut self) {
    // This won't be run by Skribe
}
```

### Function Signatures

Test functions can take parameters, which Skribe will automatically fuzz with randomized values. This is one of Skribe's key features: property testing through automatic input generation.

**Parameters are fuzzed automatically:**

```rust
pub fn test_with_parameter(&mut self, x: U256, y: U256) {
    // x and y will be fuzzed with random values
    // Skribe will generate many different combinations
    let sum = x.checked_add(y);
    assert!(sum.is_some() || (x > U256::MAX - y));
}
```

Skribe will generate many different input combinations for `x` and `y`, helping you discover edge cases and validate that your contract behaves correctly across a wide range of inputs.

**Supported parameter types:**

Common types you can use as test function parameters include:
- `U256` for unsigned 256-bit integers;
- `Address` for Ethereum addresses;
- `bool` for boolean values;
- `Bytes` or `Vec<u8>` for byte arrays;
- Other Stylus SDK types.

**Using `assume()` to filter inputs:**

If you need to restrict the range of fuzzed inputs, use the `assume()` cheatcode:

```rust
pub fn test_restricted_range(&mut self, x: U256) {
    // Only test with values in a specific range
    cheat().assume(&mut *self, x > U256::from(100) && x < U256::from(1000)).unwrap();
    
    // Test logic that assumes x is in the range (100, 1000)
}
```

When `assume()` is called with a false condition, Skribe discards that input and generates a new one, effectively filtering the input space.

### Assertions

Use standard Rust assertions to verify expected behavior. A panic in a test function is considered a test failure, so any assertion that fails will cause the test to fail.

**Common assertion patterns:**

```rust
// Equality assertion
assert_eq!(expected, actual);

// Boolean condition assertion
assert!(condition);

// Custom panic message
assert!(condition, "Custom error message");

// Unwrap with expectation
let result = some_operation().expect("Operation should succeed");
```

**Example:**

```rust
pub fn test_counter_increment(&mut self, initial: U256) {
    let counter = ICounter::new(self.counter_address.get());
    
    counter.set_number(&mut *self, initial).unwrap();
    counter.increment(&mut *self).unwrap();
    
    let new_value = counter.number(self).unwrap();
    assert_eq!(new_value, initial + U256::from(1));
}
```

When an assertion fails, Skribe will report the failing input values, making it easy to reproduce and debug the issue.

## Example Test Contracts

[Skribe's repository](https://github.com/runtimeverification/skribe) includes several example test contracts demonstrating different testing patterns and cross-testing scenarios. These examples serve as reference implementations and are excellent starting points for understanding how to structure your own tests.

**Available examples:**

- **`test-hello-world`**: Basic Stylus test contract that demonstrates deploying and interacting with a Stylus contract. Shows fundamental testing patterns and cheatcode usage;

- **`test-foundry-from-stylus`**: Stylus test contract that deploys and interacts with Solidity contracts. Demonstrates cross-testing from Stylus to Solidity, including loading Solidity bytecode and making cross-contract calls;

- **`test-stylus-from-foundry`**: Solidity test contract that deploys and interacts with Stylus contracts. Demonstrates cross-testing from Solidity to Stylus, including WASM deployment and interaction patterns;

- **`test-foundry-simple`**: Standard Foundry test contracts showing traditional Solidity testing patterns. Useful for understanding how Foundry conventions work within Skribe;

- **`test-cheatcodes`**: Examples demonstrating various cheatcode operations available in Stylus tests, including `deal()`, `warp()`, `roll()`, `store()`, and `load()`.

These examples are located in `src/tests/integration/data/contracts/` and serve as comprehensive reference implementations for common testing patterns. We recommend exploring these examples to understand the different ways you can structure and write tests with Skribe.

## Running Tests

For more information on running tests and available usage options, see the [Running Tests](../4%20-%20usage/README.md) guide.
