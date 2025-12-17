# Cheatcodes

Cheatcodes are special functions that allow you to manipulate blockchain state, control execution flow, and access testing utilities during test execution. Skribe provides cheatcodes for both Solidity and Rust test contracts.

## Solidity Cheatcodes

In Solidity tests, cheatcodes are accessed through the `vm` object, following standard Foundry conventions. Skribe supports all standard Foundry cheatcodes.

For detailed documentation on these cheatcodes, including parameters, return values, and usage examples, refer to the [Foundry Cheatcodes documentation](https://book.getfoundry.sh/cheatcodes/).

## Rust Cheatcodes

In Rust test contracts, cheatcodes are accessed through the `skribe::cheat()` function, which returns an interface to all available cheatcode operations. While the functionality matches Foundry's cheatcodes, the syntax and usage patterns are specific to Rust and the Stylus SDK.

### Accessing Cheatcodes

Import and use the `cheat()` function:

```rust
use skribe::cheat;

pub fn test_example(&mut self) {
    cheat().deal(&mut *self, address, balance).unwrap();
}
```

All cheatcode methods require `&mut self` as the first parameter and return a `Result` type that should be unwrapped or handled appropriately.

### Input Filtering

**`assume(&mut self, bool condition) -> Result<()>`**

Discards the current fuzz input if the condition is false and generates a new one. Use this to restrict the input space for property testing.

```rust
pub fn test_restricted_range(&mut self, x: U256) {
    cheat().assume(&mut *self, x > U256::from(100) && x < U256::from(1000)).unwrap();
    // Test logic that assumes x is in range (100, 1000)
}
```

### Account Manipulation

**`deal(&mut self, address account, U256 newBalance) -> Result<()>`**

Sets the balance of an address to the specified value.

```rust
let alice = address!("AABBCCDDEEFF0011223344556677889900AABBCC");
let balance = U256::from(1000000000000000000u128); // 1 ether
cheat().deal(&mut *self, alice, balance).unwrap();
```

**`prank(&mut self, address msgSender) -> Result<()>`**

Sets `msg.sender` for the next call only.

```rust
cheat().prank(&mut *self, alice).unwrap();
// Next call will have alice as msg.sender
```

**`startPrank(&mut self, address msgSender) -> Result<()>`**

Sets `msg.sender` for all subsequent calls until `stopPrank()` is called.

```rust
cheat().startPrank(&mut *self, alice).unwrap();
// All subsequent calls will have alice as msg.sender
cheat().stopPrank(&mut *self).unwrap();
```

**`stopPrank(&mut self) -> Result<()>`**

Resets `msg.sender` to the test contract's address.

### Block Environment

**`warp(&mut self, U256 newTimestamp) -> Result<()>`**

Sets `block.timestamp` to the specified value.

```rust
let timestamp = U256::from(1234567890u64);
cheat().warp(&mut *self, timestamp).unwrap();
```

**`roll(&mut self, U256 newBlockNumber) -> Result<()>`**

Sets `block.number` to the specified value.

```rust
let block_number = U256::from(1000u64);
cheat().roll(&mut *self, block_number).unwrap();
```

**`fee(&mut self, U256 newBasefee) -> Result<()>`**

Sets `block.basefee` to the specified value.

```rust
let basefee = U256::from(100000000000u128); // 100 gwei
cheat().fee(&mut *self, basefee).unwrap();
```

### Storage Manipulation

**`load(&mut self, address target, bytes32 slot) -> Result<bytes32>`**

Reads a storage slot from an address. Returns `bytes32` as `FixedBytes`.

```rust
let target = address!("...");
let slot = FixedBytes::from(U256::ZERO);
let value = cheat().load(&mut *self, target, slot).unwrap();
let value_u256: U256 = value.into();
```

**`store(&mut self, address target, bytes32 slot, bytes32 value) -> Result<()>`**

Writes a value to a storage slot at an address.

```rust
let target = address!("...");
let slot = FixedBytes::from(U256::ZERO);
let value = FixedBytes::from(U256::from(123));
cheat().store(&mut *self, target, slot, value).unwrap();
```

### Code Manipulation

**`etch(&mut self, address target, &[u8] newRuntimeBytecode) -> Result<()>`**

Sets the runtime bytecode of an address.

```rust
let target = address!("...");
let bytecode = vec![0x60, 0x00, 0x52, ...];
cheat().etch(&mut *self, target, &bytecode).unwrap();
```

### File Operations

**`readFile(&mut self, string path) -> Result<String>`**

Reads a file as a string. The path is relative to the project root.

```rust
let content = cheat()
    .read_file(&mut *self, "data/config.json".to_string())
    .unwrap();
```

**`readFileBinary(&mut self, string path) -> Result<Vec<u8>>`**

Reads a file as binary bytes. The path is relative to the project root. This is commonly used to load WASM bytecode for deploying Stylus contracts.

```rust
let bytecode = cheat()
    .read_file_binary(&mut *self, "wasm/contract.wasm".to_string())
    .unwrap();
```

### Expectation Functions

**`expectRevert(&mut self) -> Result<()>`**

Expects the next call to revert with any revert data.

```rust
cheat().expect_revert(&mut *self).unwrap();
// Next call should revert
```

**`expectEmit(&mut self) -> Result<()>`**

Prepares to check that the next call emits an event with matching topics and data. Call this before emitting an event, then make the call that should emit it.

```rust
cheat().expect_emit(&mut *self).unwrap();
// Emit event, then make call
```

### Cryptographic Functions

**`sign(&mut self, U256 privateKey, bytes32 digest) -> Result<(u8, bytes32, bytes32)>`**

Signs a digest with a private key using the secp256k1 curve. Returns `(v, r, s)` signature components.

```rust
let private_key = U256::from(123);
let digest = FixedBytes::from([0u8; 32]);
let (v, r, s) = cheat().sign(&mut *self, private_key, digest).unwrap();
```

**`addr(&mut self, U256 privateKey) -> Result<Address>`**

Gets the address corresponding to a private key.

```rust
let private_key = U256::from(123);
let key_address = cheat().addr(&mut *self, private_key).unwrap();
```

### Query Functions

**`getNonce(&mut self, address account) -> Result<u64>`**

Gets the nonce of an account.

```rust
let account = address!("...");
let nonce = cheat().get_nonce(&mut *self, account).unwrap();
```

**`projectRoot(&mut self) -> Result<String>`**

Gets the path of the current project root.

```rust
let root = cheat().project_root(&mut *self).unwrap();
```

### Assertion Functions

**`assertTrue(&mut self, bool condition) -> Result<()>`**

Asserts that the given condition is true. Panics if false.

```rust
cheat().assert_true(&mut *self, condition).unwrap();
```

## Additional Notes

Both Solidity and Rust cheatcodes provide equivalent functionality. The main difference is the syntax: Solidity uses the `vm` object while Rust uses the `cheat()` function interface with Rust specific types and error handling.

For examples of cheatcode usage in Rust tests, see the test contracts in `src/tests/integration/data/contracts/`, particularly `test-cheatcodes` which demonstrates various cheatcode operations.
