# Examples

This page showcases practical examples of WebAssembly components built with Wassette. Each example demonstrates different capabilities and programming languages, helping you understand what's possible and how to build your own components.

## Language Support Matrix

Wassette components can be written in any language that compiles to WebAssembly with Component Model support:

| Language | Support Level | Tooling | Example |
|----------|--------------|---------|---------|
| **Rust** | ✅ Full | `cargo component` | [filesystem-rs](https://github.com/microsoft/wassette/tree/main/examples/filesystem-rs), [fetch-rs](https://github.com/microsoft/wassette/tree/main/examples/fetch-rs) |
| **JavaScript** | ✅ Full | `componentize-js` | [time-server-js](https://github.com/microsoft/wassette/tree/main/examples/time-server-js), [get-weather-js](https://github.com/microsoft/wassette/tree/main/examples/get-weather-js) |
| **Python** | ✅ Full | `componentize-py` | [eval-py](https://github.com/microsoft/wassette/tree/main/examples/eval-py) |
| **Go** | ✅ Full | TinyGo + `wit-bindgen` | [gomodule-go](https://github.com/microsoft/wassette/tree/main/examples/gomodule-go) |
| **C/C++** | 🔄 Experimental | `wit-bindgen` | Coming soon |
| **C#/.NET** | 🔄 Planned | `wit-bindgen` | Coming soon |

## Example Components

### Time Server (JavaScript)

**Location:** `examples/time-server-js/`

**Description:** A simple time server that returns the current date and time.

**Language:** JavaScript

**Tools Provided:**
- `get-current-time`: Returns the current timestamp as a string

**Load Command:**
```text
Please load the time component from oci://ghcr.io/microsoft/time-server-js:latest
```

**Usage:**
```text
What is the current time?
```

**Key Features:**
- Minimal implementation showcasing basic component structure
- No external dependencies
- Perfect starting point for learning component development

**Learn More:** [Development Guide - JavaScript](./development/javascript.md)

---

### Weather API Client (JavaScript)

**Location:** `examples/get-weather-js/`

**Description:** Fetches current weather data from the Open-Meteo API for a given location.

**Language:** JavaScript

**Tools Provided:**
- `get-weather`: Retrieves weather information for a city

**Permissions Required:**
- Network access to `api.open-meteo.com`

**Load Command:**
```text
Please load the weather component from oci://ghcr.io/microsoft/get-weather-js:latest
```

**Usage:**
```text
Grant network access to api.open-meteo.com for the weather component

What's the weather like in Seattle?
```

**Key Features:**
- Demonstrates network I/O capabilities
- Shows how to integrate with external APIs
- Handles JSON parsing and data transformation

**Learn More:** [Development Guide - JavaScript](./development/javascript.md)

---

### Filesystem Operations (Rust)

**Location:** `examples/filesystem-rs/`

**Description:** Provides file system operations including reading, writing, and listing directories.

**Language:** Rust

**Tools Provided:**
- `read-file`: Read contents of a file
- `write-file`: Write data to a file
- `list-directory`: List files in a directory
- `create-directory`: Create a new directory
- `delete-file`: Remove a file

**Permissions Required:**
- Storage access with read/write permissions for specific paths

**Load Command:**
```text
Please load the filesystem component from oci://ghcr.io/microsoft/filesystem-rs:latest
```

**Usage:**
```text
Grant the filesystem component read and write access to /tmp/test

Create a file at /tmp/test/hello.txt with the content "Hello, Wassette!"

Read the file /tmp/test/hello.txt

List all files in /tmp/test
```

**Key Features:**
- Fine-grained file system access control
- Comprehensive file operations
- Error handling for invalid paths
- Demonstrates Rust's performance benefits

**Learn More:** [Development Guide - Rust](./development/rust.md)

---

### HTTP Fetch (Rust)

**Location:** `examples/fetch-rs/`

**Description:** HTTP client for fetching and converting web content to various formats.

**Language:** Rust

**Tools Provided:**
- `fetch`: HTTP GET request with format conversion
- `fetch-json`: Fetch and parse JSON responses
- `fetch-text`: Fetch plain text content

**Permissions Required:**
- Network access to target hosts

**Load Command:**
```text
Please load the fetch component from oci://ghcr.io/microsoft/fetch-rs:latest
```

**Usage:**
```text
Grant network access to api.github.com for the fetch component

Fetch the GitHub API status: https://api.github.com/status
```

**Key Features:**
- Multiple content-type handling
- JSON parsing
- Custom headers support
- Error handling for network failures

**Learn More:** [Development Guide - Rust](./development/rust.md)

---

### Python Code Execution (Python)

**Location:** `examples/eval-py/`

**Description:** Secure Python code execution sandbox for running untrusted Python code.

**Language:** Python

**Tools Provided:**
- `eval-python`: Execute Python code and return results
- `eval-python-safe`: Execute with additional safety restrictions

**Permissions Required:**
- None (runs in isolated sandbox)

**Load Command:**
```text
Please load the python eval component from oci://ghcr.io/microsoft/eval-py:latest
```

**Usage:**
```text
Execute this Python code: print("Hello from Python!")

Calculate fibonacci(10) using Python
```

**Key Features:**
- Sandboxed execution environment
- Access to standard library
- Output capture
- Safety restrictions for untrusted code

**Security Note:** While this component runs in a WebAssembly sandbox, be cautious with untrusted code execution. Always review code before running.

**Learn More:** [Development Guide - Python](./development/python.md)

---

### Go Module Info (Go)

**Location:** `examples/gomodule-go/`

**Description:** Retrieves information about Go modules and packages.

**Language:** Go (TinyGo)

**Tools Provided:**
- `get-module-info`: Get details about a Go module
- `list-dependencies`: List module dependencies

**Permissions Required:**
- Network access to `proxy.golang.org`

**Load Command:**
```text
Please load the gomodule component from oci://ghcr.io/microsoft/gomodule-go:latest
```

**Usage:**
```text
Grant network access to proxy.golang.org for the gomodule component

What's the latest version of github.com/spf13/cobra?
```

**Key Features:**
- Integration with Go module proxy
- Version resolution
- Dependency analysis
- Demonstrates TinyGo capabilities

**Learn More:** [Development Guide - Go](./development/go.md)

---

## Community Components

The Wassette community has created additional components:

### QR Code Generator

**Author:** @attackordie

**Description:** Generate QR codes from text using a WebAssembly component.

**Repository:** [github.com/attackordie/qr-code-webassembly](https://github.com/attackordie/qr-code-webassembly)

**Tools Provided:**
- `generate-qr-code`: Create QR codes from input text

**Usage:**
```text
Generate a QR code for "https://microsoft.com"
```

---

## Example Use Cases

### Data Processing Pipeline

Combine multiple components to create a data processing workflow:

```text
1. Load the fetch component
2. Grant it network access to api.example.com
3. Fetch data from the API
4. Load the filesystem component
5. Grant it write access to /tmp/output
6. Save the processed data to a file
```

### Development Assistant

Build a development helper with multiple tools:

```text
1. Load the gomodule component for package lookups
2. Load the fetch component for documentation
3. Load the filesystem component for reading project files
4. Ask: "What dependencies does my go.mod file use and are they up to date?"
```

### Data Analysis

Create an analysis pipeline:

```text
1. Load the python eval component
2. Load the filesystem component with read access to your data directory
3. Ask: "Read the CSV file at /path/to/data.csv and calculate summary statistics"
```

## Building Your Own Component

Ready to create your own component? Choose your preferred language:

- **[Rust](./development/rust.md)** - Best performance, comprehensive tooling
- **[JavaScript](./development/javascript.md)** - Quick prototyping, familiar syntax
- **[Python](./development/python.md)** - Easy development, great for scripts
- **[Go](./development/go.md)** - Strong typing, excellent standard library

Each guide includes:
- Development environment setup
- Step-by-step implementation
- Building and testing
- Publishing to OCI registries
- Best practices and common patterns

## Component Design Patterns

### Stateless Functions

Simple, stateless operations that take input and return output:

```wit
world my-tool {
    export process: func(input: string) -> string;
}
```

**Example:** Time server, calculators, formatters

### Resource Management

Components that manage external resources with explicit lifecycle:

```wit
world database-client {
    export connect: func(url: string) -> result<connection>;
    export query: func(conn: connection, sql: string) -> result<list<record>>;
    export close: func(conn: connection);
}
```

**Example:** Database clients, file handles, network connections

### Batch Operations

Processing multiple items efficiently:

```wit
world batch-processor {
    export process-batch: func(items: list<string>) -> list<result<string>>;
}
```

**Example:** Image processing, data transformation, bulk operations

### Configuration-Based

Tools with configurable behavior:

```wit
world configurable-tool {
    export configure: func(options: config-options);
    export execute: func(input: string) -> result<output>;
}
```

**Example:** Format converters, API clients with different backends

## Testing Your Components

### Local Testing

Before publishing, test your component locally:

```bash
# Load from local filesystem
wassette component load file:///absolute/path/to/your-component.wasm

# Test the tools
wassette component list
```

### Integration Testing

Use the MCP Inspector to test your component with an AI agent:

```bash
# Start Wassette with inspector
npx @modelcontextprotocol/inspector wassette serve --stdio
```

This provides a web interface for interactive testing.

## Publishing Components

Once your component is ready:

1. **Build for production:**
   ```bash
   # Optimize your component
   wasm-opt -O3 component.wasm -o component-optimized.wasm
   ```

2. **Test thoroughly:**
   - Unit tests in your source language
   - Integration tests with Wassette
   - Permission verification

3. **Publish to OCI registry:**
   ```bash
   # Tag and push to GitHub Container Registry
   crane push component-optimized.wasm ghcr.io/your-org/your-component:latest
   ```

4. **Document your component:**
   - What tools does it provide?
   - What permissions does it need?
   - What are the input/output formats?
   - Include usage examples

5. **Share with the community:**
   - Add to the component registry
   - Submit a PR to showcase in this documentation
   - Share in the Discord community

## Contributing Examples

Have a great component example? We'd love to include it:

1. **Prepare your example:**
   - Clear documentation
   - Well-structured code
   - Comprehensive README
   - Build instructions

2. **Submit to the repository:**
   - Fork the [Wassette repository](https://github.com/microsoft/wassette)
   - Add your example to the `examples/` directory
   - Include it in this examples documentation
   - Submit a pull request

3. **Community showcase:**
   - Share in the Discord #wassette channel
   - Add to the community components section
   - Blog about your component

## Resources

- **Documentation:** Complete guides for each language in [Development](./development.md)
- **Source Code:** All examples at [github.com/microsoft/wassette/tree/main/examples](https://github.com/microsoft/wassette/tree/main/examples)
- **Component Registry:** Browse available components with `search-components` tool
- **Discord:** Join the conversation in the [#wassette channel](https://discord.gg/microsoft-open-source)
- **GitHub:** Report issues or request features at [github.com/microsoft/wassette](https://github.com/microsoft/wassette)
