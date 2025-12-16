# Skribe

Skribe is a fuzzer and property testing framework for Stylus smart contracts. It enables developers to write comprehensive test suites that automatically discover and validate contract behavior through fuzzing.

Stylus is Arbitrum's smart contract platform that allows developers to write contracts in Rust and compile them to WebAssembly. Leveraging the K framework's formal semantics, Skribe executes these contracts in a controlled testing environment, providing deep integration with the Stylus execution model.

With Skribe, you write test contracts in Rust following a simple convention. Test functions are automatically discovered and executed with randomized inputs, helping you find edge cases, validate invariants, and ensure your contracts behave correctly under various conditions. Skribe supports testing individual contracts, multi contract interactions, and integration with Foundry based test suites. We highlight that the Foundry tests can interact with deployed WASM contracts, and the test written in Rust can interact with the deployed Solidity contracts.

The framework provides cheatcodes for manipulating blockchain state (available for both Rust and Solidity), reading files, and controlling test execution flow. This makes it possible to test complex scenarios including contract deployments, state transitions, and cross contract calls.

Whether you are building a new Stylus contract or maintaining an existing one, Skribe helps ensure your code is robust, secure, and ready for production deployment.

To get started, please refer to the Quick Start guide.