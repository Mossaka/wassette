# Development Setup

This guide will help you set up your development environment for contributing to Wassette or building WebAssembly components locally. Whether you're working on the Wassette runtime itself or creating new components, this guide covers everything you need.

## Table of Contents

- [Setting Up for Wassette Development](#setting-up-for-wassette-development)
- [Setting Up for Component Development](#setting-up-for-component-development)
- [Development Workflow](#development-workflow)
- [Testing](#testing)
- [Debugging](#debugging)
- [Contributing](#contributing)

## Setting Up for Wassette Development

If you want to contribute to Wassette itself or build it from source:

### Prerequisites

1. **Rust Toolchain** (1.75.0 or later)
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   source ~/.cargo/env
   ```

2. **Git**
   ```bash
   # Ubuntu/Debian
   sudo apt-get install git

   # macOS
   brew install git

   # Windows
   winget install Git.Git
   ```

3. **Just** (command runner - optional but recommended)
   ```bash
   cargo install just
   ```

### Clone the Repository

```bash
git clone https://github.com/microsoft/wassette.git
cd wassette
```

### Build from Source

```bash
# Build debug version
cargo build

# Build release version (optimized)
cargo build --release

# The binary will be at:
# - Debug: target/debug/wassette
# - Release: target/release/wassette
```

### Using the Justfile

Wassette includes a `Justfile` with common development tasks:

```bash
# Show available commands
just --list

# Build the project
just build

# Run tests
just test

# Run clippy (linter)
just clippy

# Format code
just fmt

# Run the server in development mode
just run

# Run examples
just examples

# Build documentation
just docs

# Clean build artifacts
just clean
```

### Project Structure

```
wassette/
├── crates/
│   ├── component2json/    # Component schema extraction
│   ├── mcp-server/         # MCP server implementation
│   ├── policy/             # Security policy engine
│   └── wassette/           # Main runtime and CLI
├── docs/                   # Documentation (mdBook)
├── examples/               # Example components
├── src/                    # Main entry point
├── tests/                  # Integration tests
└── Justfile               # Development commands
```

## Setting Up for Component Development

If you want to build WebAssembly components to use with Wassette:

### Choose Your Language

- **[Rust](./development/rust.md)** - Comprehensive guide for Rust components
- **[JavaScript](./development/javascript.md)** - Guide for JavaScript/TypeScript components
- **[Python](./development/python.md)** - Guide for Python components  
- **[Go](./development/go.md)** - Guide for Go components

Each guide includes:
- Language-specific toolchain setup
- Creating a new component project
- Defining interfaces with WIT
- Building and testing
- Publishing components

### Common Tools for Component Development

Regardless of language, you'll need:

1. **WebAssembly Binary Toolkit (WABT)** - For inspecting and debugging
   ```bash
   # Ubuntu/Debian
   sudo apt-get install wabt

   # macOS
   brew install wabt

   # Or build from source
   git clone https://github.com/WebAssembly/wabt
   cd wabt && make
   ```

2. **wasm-tools** - Component Model utilities
   ```bash
   cargo install wasm-tools
   ```

3. **wasm-opt** (optional) - Optimization tool
   ```bash
   # Ubuntu/Debian
   sudo apt-get install binaryen

   # macOS
   brew install binaryen
   ```

## Development Workflow

### 1. Making Changes

```bash
# Create a feature branch
git checkout -b feature/my-feature

# Make your changes
# ...

# Format code
cargo fmt

# Run linter
cargo clippy --all-targets --all-features

# Run tests
cargo test
```

### 2. Testing Changes

```bash
# Run all tests
just test

# Run specific test
cargo test test_name

# Run with verbose output
cargo test -- --nocapture

# Run integration tests
cargo test --test '*'
```

### 3. Building Documentation

```bash
# Build and serve documentation locally
just docs

# Or manually
cd docs
mdbook serve --open
```

The documentation will be available at `http://localhost:3000`.

### 4. Running Examples

```bash
# Build all examples
just examples

# Or build specific example
cd examples/time-server-js
npm install
npm run build
```

## Testing

### Unit Tests

Unit tests are located alongside the code in `src/` and `crates/`:

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_something() {
        // Test implementation
    }
}
```

Run unit tests:
```bash
cargo test --lib
```

### Integration Tests

Integration tests are in the `tests/` directory:

```bash
# Run all integration tests
cargo test --test '*'

# Run specific integration test
cargo test --test integration_test_name
```

### Example Tests

Test the example components:

```bash
# Test all examples
cd examples
for dir in */; do
    cd "$dir"
    npm test || cargo test || python -m pytest || go test ./...
    cd ..
done
```

### End-to-End Tests

Test the full MCP server interaction:

```bash
# Start the server
just run &

# Run MCP inspector
npx @modelcontextprotocol/inspector --cli http://127.0.0.1:9001/sse

# Test specific operations
npx @modelcontextprotocol/inspector --cli http://127.0.0.1:9001/sse --method tools/list
```

## Debugging

### Debugging Wassette

#### Using Rust Debugger

```bash
# With lldb (macOS/Linux)
lldb target/debug/wassette
(lldb) run serve --stdio

# With gdb (Linux)
gdb target/debug/wassette
(gdb) run serve --stdio
```

#### Logging

Enable debug logging:

```bash
# Set log level
export RUST_LOG=debug
wassette serve --stdio

# Or more specific
export RUST_LOG=wassette=debug,mcp_server=trace
wassette serve --stdio
```

### Debugging Components

#### Inspect Component Schema

```bash
# View component exports
wasm-tools component wit component.wasm

# View detailed metadata
wasm-tools print component.wasm
```

#### Test Component Locally

```bash
# Load component via CLI
wassette component load file:///path/to/component.wasm

# List available tools
wassette component list

# Get component policy
wassette policy get <component-id>
```

#### MCP Inspector

Use the MCP Inspector for interactive debugging:

```bash
# Start Wassette with inspector
npx @modelcontextprotocol/inspector wassette serve --stdio
```

This provides a web interface at `http://localhost:5173` where you can:
- List available tools
- Call tools with custom parameters
- View request/response data
- Debug permission issues

### Common Issues

#### Build Failures

```bash
# Clean and rebuild
cargo clean
cargo build

# Update dependencies
cargo update
```

#### Test Failures

```bash
# Run with backtrace
RUST_BACKTRACE=1 cargo test

# Run with full output
cargo test -- --nocapture --test-threads=1
```

#### Component Loading Issues

```bash
# Verify component format
wasm-tools validate component.wasm

# Check component exports match expectations
wasm-tools component wit component.wasm

# Test with detailed logging
RUST_LOG=debug wassette component load file:///path/to/component.wasm
```

## Contributing

### Code Style

Wassette follows standard Rust conventions:

```bash
# Format code
cargo fmt

# Check formatting without applying
cargo fmt -- --check

# Run clippy
cargo clippy --all-targets --all-features

# Fix clippy warnings automatically (when safe)
cargo clippy --fix
```

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
type(scope): subject

body

footer
```

Examples:
```
feat(cli): add component unload command

Implements the ability to unload components via CLI
without requiring MCP server connection.

Closes #123
```

```
fix(policy): correct storage permission validation

The storage permission validator was incorrectly rejecting
valid file:// URIs with query parameters.

Fixes #456
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Test additions or changes
- `chore`: Build process or tooling changes

### Pull Request Process

1. **Fork the repository** and create your branch from `main`

2. **Make your changes** following the code style guidelines

3. **Add tests** for new functionality

4. **Update documentation** as needed

5. **Run the test suite:**
   ```bash
   just test
   cargo clippy
   cargo fmt -- --check
   ```

6. **Commit your changes** with clear, descriptive messages

7. **Push to your fork** and submit a pull request

8. **Respond to review feedback** and make requested changes

9. **Ensure CI passes** on your pull request

### Changelog Updates

When making changes:

1. **Update `CHANGELOG.md`** in the `[Unreleased]` section
2. **Categorize under the appropriate section:**
   - **Added** for new features
   - **Changed** for changes in existing functionality
   - **Deprecated** for soon-to-be removed features
   - **Removed** for now removed features
   - **Fixed** for any bug fixes
   - **Security** for vulnerability fixes
3. **Reference the pull request:** `([#123](https://github.com/microsoft/wassette/pull/123))`

Example:
```markdown
## [Unreleased]

### Added
- CLI command for component management ([#123](https://github.com/microsoft/wassette/pull/123))

### Fixed
- Storage permission validation for query parameters ([#456](https://github.com/microsoft/wassette/pull/456))
```

## Development Tools

### Recommended IDE Setup

#### VS Code

Install these extensions:
- **rust-analyzer** - Rust language support
- **CodeLLDB** - Debugging support
- **Even Better TOML** - TOML file support
- **crates** - Cargo dependency management

#### IntelliJ IDEA / CLion

- Install the **Rust plugin**
- Configure Rust toolchain in settings
- Enable Clippy integration

### Performance Profiling

```bash
# Build with debug symbols
cargo build --release --features debug

# Profile with perf (Linux)
perf record --call-graph dwarf target/release/wassette serve --stdio
perf report

# Profile with Instruments (macOS)
# Run with Xcode Instruments -> Time Profiler
```

### Memory Debugging

```bash
# Use valgrind (Linux)
valgrind --leak-check=full target/debug/wassette serve --stdio

# Use Address Sanitizer
RUSTFLAGS="-Z sanitizer=address" cargo +nightly build
target/debug/wassette serve --stdio
```

## Resources

- **Contributing Guide:** [CONTRIBUTING.md](https://github.com/microsoft/wassette/blob/main/CONTRIBUTING.md)
- **Code of Conduct:** [CODE_OF_CONDUCT.md](https://github.com/microsoft/wassette/blob/main/CODE_OF_CONDUCT.md)
- **Issue Tracker:** [github.com/microsoft/wassette/issues](https://github.com/microsoft/wassette/issues)
- **Discussions:** [github.com/microsoft/wassette/discussions](https://github.com/microsoft/wassette/discussions)
- **Discord:** [#wassette channel](https://discord.gg/microsoft-open-source)

## Next Steps

- **Build a component:** Choose your language from the [component development guides](#setting-up-for-component-development)
- **Explore examples:** Check out [Examples](./examples.md) for inspiration
- **Join the community:** Connect on [Discord](https://discord.gg/microsoft-open-source)
- **Contribute:** Find [good first issues](https://github.com/microsoft/wassette/labels/good%20first%20issue) to get started
