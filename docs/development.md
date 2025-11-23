# Development Guide

This guide provides comprehensive instructions for contributors who want to build, test, and improve Wassette.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Development Environment Setup](#development-environment-setup)
- [Building from Source](#building-from-source)
- [Running Tests](#running-tests)
- [Development Workflows](#development-workflows)
- [Debugging](#debugging)
- [Code Style and Standards](#code-style-and-standards)
- [Contributing Guidelines](#contributing-guidelines)

## Prerequisites

Before you begin development, ensure you have:

1. **Rust toolchain** (1.75.0 or later):
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   source ~/.cargo/env
   ```

2. **Just command runner** (optional but recommended):
   ```bash
   # macOS
   brew install just
   
   # Linux
   cargo install just
   
   # Or download from https://github.com/casey/just
   ```

3. **Git** for version control:
   ```bash
   git --version
   ```

4. **MCP Inspector** for testing (optional):
   ```bash
   npm install -g @modelcontextprotocol/inspector
   ```

## Development Environment Setup

### 1. Clone the Repository

```bash
git clone https://github.com/microsoft/wassette.git
cd wassette
```

### 2. Install Development Dependencies

```bash
# Install Rust toolchain components
rustup component add rustfmt clippy

# Install WASI target for building examples
rustup target add wasm32-wasip2

# Verify installation
cargo --version
rustc --version
```

### 3. Initial Build

Build the project to ensure everything is set up correctly:

```bash
# Using Just (recommended)
just build

# Or using Cargo directly
cargo build
```

## Building from Source

### Development Build

For faster compilation during development:

```bash
# Build debug version
just build
# or
cargo build

# Run the binary
./target/debug/wassette --help
```

### Release Build

For optimized production builds:

```bash
# Build release version
just release
# or
cargo build --release

# Run the optimized binary
./target/release/wassette --help
```

### Building Specific Crates

Wassette is organized into multiple crates:

```bash
# Build only the MCP server crate
cargo build -p mcp-server

# Build the component2json utility
cargo build -p component2json

# Build the policy crate
cargo build -p policy
```

### Build Options

```bash
# Clean build artifacts
just clean
# or
cargo clean

# Build with all features
cargo build --all-features

# Build documentation
cargo doc --no-deps --open
```

## Running Tests

Wassette includes comprehensive test coverage across unit, integration, and end-to-end tests.

### Quick Test Commands

```bash
# Run all tests
just test

# Run tests with output
just test-verbose

# Run specific test file
cargo test --test cli_integration_test

# Run specific test function
cargo test test_load_component_from_file
```

### Test Categories

#### Unit Tests

Test individual functions and modules:

```bash
# Run unit tests only
cargo test --lib

# Run tests for specific crate
cargo test -p policy

# Run with all features enabled
cargo test --all-features
```

#### Integration Tests

Test component interactions:

```bash
# Run all integration tests
cargo test --tests

# Run specific integration test suite
cargo test --test transport_integration_test
cargo test --test oci_integration_test
cargo test --test grant_permission_integration_test
```

#### End-to-End Tests

Test complete workflows:

```bash
# Run E2E tests (requires built components)
cargo test --test cli_integration_test
cargo test --test structured_output_integration_test
```

### Test Coverage

Generate test coverage reports:

```bash
# Install tarpaulin (first time only)
cargo install cargo-tarpaulin

# Generate coverage report
cargo tarpaulin --out Html --output-dir coverage
```

### Example Component Tests

Test example WebAssembly components:

```bash
# Test all examples
just test-examples

# Build and test specific example
cd examples/time-server-js
just build
just test
```

## Development Workflows

### Using Justfile Commands

The repository includes a Justfile with common development tasks:

```bash
# View all available commands
just --list

# Development commands
just build          # Build debug version
just test           # Run all tests
just fmt            # Format code
just lint           # Run clippy lints
just check          # Check code without building

# Release commands
just release        # Build release version
just install        # Install locally

# Maintenance
just clean          # Remove build artifacts
just update-deps    # Update dependencies
```

### Running the Server

Start the Wassette MCP server for development:

```bash
# Start with stdio transport (for MCP clients)
just run
# or
cargo run -- serve --stdio

# Start with HTTP transport (for testing)
cargo run -- serve --http

# Start with SSE transport
cargo run -- serve --sse
```

### Component Management

Work with components during development:

```bash
# Load a component from file
cargo run -- component load file://./examples/time-server-js/time.wasm

# Load from OCI registry
cargo run -- component load oci://ghcr.io/yoshuawuyts/time:latest

# List loaded components
cargo run -- component list --output-format table

# Unload a component
cargo run -- component unload <component-id>
```

## Debugging

### MCP Inspector

Use the MCP Inspector to debug MCP protocol interactions:

```bash
# Start Wassette with stdio transport
cargo run -- serve --stdio &

# Connect inspector to the running server
npx @modelcontextprotocol/inspector
```

In the inspector:
1. Connect to `127.0.0.1:9001` (or stdio)
2. View available tools
3. Test tool calls with custom inputs
4. Inspect responses and errors

### Logging

Enable debug logging for detailed output:

```bash
# Set log level
export RUST_LOG=debug
cargo run -- serve --stdio

# Log specific modules
export RUST_LOG=wassette=debug,mcp_server=trace
cargo run -- serve --stdio

# Log to file
cargo run -- serve --stdio 2>&1 | tee wassette.log
```

### Debugging Tests

Run tests with output and backtraces:

```bash
# Show test output
cargo test -- --nocapture

# Enable backtraces
RUST_BACKTRACE=1 cargo test

# Full backtrace
RUST_BACKTRACE=full cargo test

# Debug specific test
cargo test test_name -- --nocapture --exact
```

### Using GDB/LLDB

Debug with native debuggers:

```bash
# Build with debug symbols
cargo build --debug

# Run with GDB (Linux)
gdb --args ./target/debug/wassette serve --stdio

# Run with LLDB (macOS)
lldb -- ./target/debug/wassette serve --stdio
```

### Component Inspection

Inspect WebAssembly components:

```bash
# Build component2json tool
cargo build -p component2json

# Inspect a component
./target/debug/component2json path/to/component.wasm

# Pretty print JSON output
./target/debug/component2json component.wasm | jq .
```

## Code Style and Standards

### Formatting

Wassette uses `rustfmt` with nightly features:

```bash
# Format all code
just fmt
# or
cargo +nightly fmt

# Check formatting without changing files
cargo +nightly fmt -- --check
```

The project uses a custom `rustfmt.toml` configuration for consistent formatting.

### Linting

Run Clippy for code quality checks:

```bash
# Run all lints
just lint
# or
cargo clippy -- -D warnings

# Run with all features
cargo clippy --all-features -- -D warnings

# Auto-fix issues (where possible)
cargo clippy --fix
```

### Copyright Headers

All Rust files must include the Microsoft copyright header:

```rust
// Copyright (c) Microsoft Corporation.
// Licensed under the MIT license.
```

Apply copyright headers automatically:

```bash
./scripts/copyright.sh
```

### Code Quality Checks

Before submitting code:

```bash
# Run the complete CI pipeline locally
just ci-check

# Or run checks individually
cargo +nightly fmt -- --check  # Formatting
cargo clippy -- -D warnings    # Linting
cargo test                     # Tests
cargo build --release          # Release build
```

### Commit Messages

Follow conventional commit format:

```
<type>(<scope>): <subject>

<body>

Signed-off-by: Your Name <your.email@example.com>
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `refactor`: Code refactoring
- `test`: Test additions/changes
- `chore`: Build/tooling changes

Example:
```
feat(cli): add memory permission management commands

Add new CLI commands for granting and revoking memory permissions
to WebAssembly components, enabling fine-grained resource control.

Signed-off-by: Jane Developer <jane@example.com>
```

## Contributing Guidelines

### Development Process

1. **Fork and Branch**:
   ```bash
   # Fork on GitHub, then clone
   git clone https://github.com/YOUR_USERNAME/wassette.git
   cd wassette
   
   # Create feature branch
   git checkout -b feature/my-new-feature
   ```

2. **Make Changes**:
   - Write code following style guidelines
   - Add tests for new functionality
   - Update documentation
   - Add copyright headers

3. **Test Thoroughly**:
   ```bash
   just test
   just lint
   just fmt
   ```

4. **Update CHANGELOG**:
   Add entry to `CHANGELOG.md` under `[Unreleased]` section following [Keep a Changelog](https://keepachangelog.com/) format.

5. **Commit and Push**:
   ```bash
   git add .
   git commit -s -m "feat: add new feature"
   git push origin feature/my-new-feature
   ```

6. **Create Pull Request**:
   - Open PR on GitHub
   - Fill in PR template
   - Link related issues
   - Wait for CI checks and review

### Review Process

Pull requests require:
- All CI checks passing
- Code review approval
- Updated tests
- Updated documentation
- CHANGELOG entry

### Getting Help

- **Issues**: [GitHub Issues](https://github.com/microsoft/wassette/issues)
- **Discussions**: [GitHub Discussions](https://github.com/microsoft/wassette/discussions)
- **Discord**: [#wassette channel](https://discord.gg/microsoft-open-source)

## Useful Commands Reference

### Build Commands
```bash
just build              # Debug build
just release            # Release build
just clean              # Clean artifacts
cargo build -p <crate>  # Build specific crate
```

### Test Commands
```bash
just test               # Run all tests
just test-verbose       # Tests with output
cargo test <name>       # Run specific test
RUST_LOG=debug cargo test  # Tests with logging
```

### Code Quality
```bash
just fmt                # Format code
just lint               # Run lints
cargo clippy --fix      # Auto-fix issues
./scripts/copyright.sh  # Add headers
```

### Development Server
```bash
just run                # Start server
cargo run -- serve --stdio  # Stdio transport
cargo run -- serve --http  # HTTP transport
cargo run -- component list  # List components
```

### Component Tools
```bash
cargo build -p component2json  # Build inspector
./target/debug/component2json component.wasm  # Inspect
npx @modelcontextprotocol/inspector  # MCP inspector
```

## Additional Resources

- [Contributing Guide](../CONTRIBUTING.md)
- [Code of Conduct](../CODE_OF_CONDUCT.md)
- [Architecture Documentation](./design/architecture.md)
- [Writing Rust Components](./development/rust.md)
- [GitHub Agentic Workflows](./agentic-workflows.md)

---

*This guide is maintained by the Wassette community. Contributions and improvements are welcome!*
