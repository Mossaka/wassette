# Examples Catalog

This catalog provides an overview of all available Wassette examples, organized by programming language and functionality. Each example demonstrates how to build WebAssembly Components that can be loaded and used as tools by AI agents through Wassette.

## Quick Start

All examples are available in the [`examples/` directory](https://github.com/microsoft/wassette/tree/main/examples) of the repository. Each example includes:

- Complete source code
- Build instructions
- README with usage examples
- WIT interface definitions
- Required dependencies

## Examples by Language

### JavaScript/TypeScript Examples

JavaScript examples use [`jco`](https://github.com/bytecodealliance/jco) (JavaScript Component Tools) to build WebAssembly Components.

#### Time Server
**Location**: [`examples/time-server-js/`](https://github.com/microsoft/wassette/tree/main/examples/time-server-js)

A simple component that provides current time information.

**Features**:
- Returns current date and time
- Demonstrates basic component structure
- No external dependencies
- Perfect for learning Wassette basics

**WIT Interface**:
```wit
package local:time-server;

interface time {
    get-current-time: func() -> string;
}

world time-server {
    export time;
}
```

**Usage**:
```text
Load oci://ghcr.io/microsoft/time-server-js:latest
What is the current time?
```

**OCI Registry**: `oci://ghcr.io/microsoft/time-server-js:latest`

---

#### Weather Service
**Location**: [`examples/get-weather-js/`](https://github.com/microsoft/wassette/tree/main/examples/get-weather-js)

Fetches weather information for a given city using the OpenWeather API.

**Features**:
- Network access to weather API
- Environment variable configuration (API key)
- Error handling with result types
- Real-world API integration example

**Requirements**:
- OpenWeather API key (set via secrets or environment)
- Network permission for `api.openweathermap.org`

**WIT Interface**:
```wit
package local:weather;

world weather-service {
    import wasi:config/store@0.2.0-draft;
    export get-weather: func(city: string) -> result<string, string>;
}
```

**Setup**:
```bash
# Set API key
wassette secret set weather-component OPENWEATHER_API_KEY your-key-here

# Grant network permission
wassette permission grant network weather-component api.openweathermap.org

# Load component
wassette component load oci://ghcr.io/microsoft/get-weather-js:latest
```

**Usage**:
```text
What's the weather in Seattle?
```

**OCI Registry**: `oci://ghcr.io/microsoft/get-weather-js:latest`

---

#### Open-Meteo Weather
**Location**: [`examples/get-open-meteo-weather-js/`](https://github.com/microsoft/wassette/tree/main/examples/get-open-meteo-weather-js)

Alternative weather service using the Open-Meteo API (no API key required).

**Features**:
- Free weather API (no authentication)
- Geographic coordinate lookup
- Network access demonstration
- Simpler than OpenWeather example

**Requirements**:
- Network permission for `api.open-meteo.com`

**Setup**:
```bash
# Grant network permission
wassette permission grant network weather-component api.open-meteo.com

# Load component
wassette component load oci://ghcr.io/microsoft/get-open-meteo-weather-js:latest
```

**Usage**:
```text
What's the temperature in Paris?
```

**OCI Registry**: `oci://ghcr.io/microsoft/get-open-meteo-weather-js:latest`

---

### Python Examples

Python examples use [`componentize-py`](https://github.com/bytecodealliance/componentize-py) to build WebAssembly Components from Python code.

#### Python Code Executor
**Location**: [`examples/eval-py/`](https://github.com/microsoft/wassette/tree/main/examples/eval-py)

A sandboxed Python code execution environment.

**Features**:
- Execute Python code safely in WebAssembly sandbox
- Captures stdout and stderr
- Timeout protection
- Error handling
- Demonstrates security benefits of Wassette

**Security Considerations**:
- Code runs in WebAssembly sandbox
- No host system access by default
- Storage/network requires explicit permissions
- Safe for untrusted code execution

**WIT Interface**:
```wit
package local:eval-py;

record eval-result {
    output: string,
    error: option<string>,
}

interface evaluator {
    eval: func(code: string) -> eval-result;
}

world eval-py {
    export evaluator;
}
```

**Usage**:
```text
Evaluate this Python code: print("Hello from WebAssembly!")
```

**Use Cases**:
- Interactive Python REPL in AI chat
- Safe code snippet testing
- Educational demonstrations
- Prototyping without local Python install

**OCI Registry**: `oci://ghcr.io/microsoft/eval-py:latest`

---

### Rust Examples

Rust examples use [`cargo-component`](https://github.com/bytecodealliance/cargo-component) to build WebAssembly Components.

#### HTTP Fetch Client
**Location**: [`examples/fetch-rs/`](https://github.com/microsoft/wassette/tree/main/examples/fetch-rs)

HTTP client for fetching and converting web content.

**Features**:
- HTTP/HTTPS requests
- Multiple response format support (JSON, HTML, plain text)
- Content type detection
- Error handling
- Demonstrates network permissions

**Requirements**:
- Network permission for target hosts

**WIT Interface**:
```wit
package local:fetch;

variant content-type {
    json,
    html,
    text,
    binary,
}

record response {
    status: u16,
    content-type: content-type,
    body: string,
}

interface http-client {
    fetch: func(url: string) -> result<response, string>;
}

world fetch {
    export http-client;
}
```

**Setup**:
```bash
# Grant network permission for specific domain
wassette permission grant network fetch-component api.github.com

# Or for testing
wassette permission grant network fetch-component httpbin.org
```

**Usage**:
```text
Fetch the content from https://api.github.com/users/microsoft
```

**OCI Registry**: `oci://ghcr.io/microsoft/fetch-rs:latest`

---

#### File System Operations
**Location**: [`examples/filesystem-rs/`](https://github.com/microsoft/wassette/tree/main/examples/filesystem-rs)

Comprehensive file system operations tool.

**Features**:
- Read files
- Write files
- List directories
- Create directories
- Delete files/directories
- File metadata
- Demonstrates storage permissions

**Requirements**:
- Storage permissions for target paths

**WIT Interface**:
```wit
package local:filesystem;

record file-info {
    name: string,
    is-file: bool,
    size: option<u64>,
}

interface operations {
    read-file: func(path: string) -> result<string, string>;
    write-file: func(path: string, content: string) -> result<_, string>;
    list-dir: func(path: string) -> result<list<file-info>, string>;
    create-dir: func(path: string) -> result<_, string>;
    delete: func(path: string) -> result<_, string>;
}

world filesystem {
    export operations;
}
```

**Setup**:
```bash
# Grant storage permission
wassette permission grant storage filesystem-component fs:///workspace/ --access read,write

# Or read-only
wassette permission grant storage filesystem-component fs:///workspace/ --access read
```

**Usage**:
```text
List the files in /workspace/
Read the contents of /workspace/notes.txt
Write "Hello World" to /workspace/test.txt
```

**Security Note**: Only grants access to explicitly permitted paths.

**OCI Registry**: `oci://ghcr.io/microsoft/filesystem-rs:latest`

---

### Go Examples

Go examples use [`tinygo`](https://tinygo.org/) with WebAssembly Component support.

#### Go Module Information
**Location**: [`examples/gomodule-go/`](https://github.com/microsoft/wassette/tree/main/examples/gomodule-go)

Retrieves information about Go modules from the Go package index.

**Features**:
- Query Go package information
- Version lookup
- Module metadata
- Network API integration
- Demonstrates Go WebAssembly support

**Requirements**:
- Network permission for `proxy.golang.org`

**WIT Interface**:
```wit
package local:gomodule;

record module-info {
    name: string,
    version: string,
    description: option<string>,
}

interface modules {
    get-module-info: func(module-path: string) -> result<module-info, string>;
    latest-version: func(module-path: string) -> result<string, string>;
}

world gomodule {
    export modules;
}
```

**Setup**:
```bash
# Grant network permission
wassette permission grant network gomodule-component proxy.golang.org
```

**Usage**:
```text
Get information about the github.com/spf13/cobra Go module
What's the latest version of golang.org/x/tools?
```

**OCI Registry**: `oci://ghcr.io/microsoft/gomodule-go:latest`

---

## Examples by Functionality

### Network Access

Examples demonstrating network permissions:

| Example | Language | API Used | OCI Path |
|---------|----------|----------|----------|
| Weather (OpenWeather) | JavaScript | openweathermap.org | `oci://ghcr.io/microsoft/get-weather-js:latest` |
| Weather (Open-Meteo) | JavaScript | open-meteo.com | `oci://ghcr.io/microsoft/get-open-meteo-weather-js:latest` |
| HTTP Fetch | Rust | Any HTTP(S) | `oci://ghcr.io/microsoft/fetch-rs:latest` |
| Go Module Info | Go | proxy.golang.org | `oci://ghcr.io/microsoft/gomodule-go:latest` |

### File System Access

Examples demonstrating storage permissions:

| Example | Language | Operations | OCI Path |
|---------|----------|------------|----------|
| File System | Rust | Read, Write, List, Delete | `oci://ghcr.io/microsoft/filesystem-rs:latest` |

### Configuration/Secrets

Examples using environment variables and secrets:

| Example | Language | Configuration | OCI Path |
|---------|----------|---------------|----------|
| Weather (OpenWeather) | JavaScript | API Key | `oci://ghcr.io/microsoft/get-weather-js:latest` |

### Code Execution

Examples demonstrating sandboxed code execution:

| Example | Language | Runtime | OCI Path |
|---------|----------|---------|----------|
| Python Eval | Python | Python interpreter | `oci://ghcr.io/microsoft/eval-py:latest` |

### Simple/Educational

Best examples for learning:

| Example | Language | Complexity | OCI Path |
|---------|----------|------------|----------|
| Time Server | JavaScript | ⭐ Basic | `oci://ghcr.io/microsoft/time-server-js:latest` |
| Weather (Open-Meteo) | JavaScript | ⭐⭐ Moderate | `oci://ghcr.io/microsoft/get-open-meteo-weather-js:latest` |

## Building Examples Locally

### Prerequisites

Install required tools for your language:

**JavaScript**:
```bash
npm install -g @bytecodealliance/jco
```

**Python**:
```bash
pip install componentize-py
```

**Rust**:
```bash
cargo install cargo-component
```

**Go**:
```bash
# Install TinyGo from https://tinygo.org/getting-started/install/
```

### Build Steps

Each example includes a `README.md` with specific build instructions. General pattern:

```bash
# Navigate to example directory
cd examples/time-server-js/

# Install dependencies (if applicable)
npm install

# Build the component
npm run build

# Test with Wassette
wassette component load file://./component.wasm
```

## Loading Examples

### From OCI Registry (Recommended)

```bash
# Load pre-built component
wassette component load oci://ghcr.io/microsoft/time-server-js:latest

# Configure permissions (if needed)
wassette permission grant network component-id api.example.com

# Use via AI agent
# Ask: "What time is it?"
```

### From Local Build

```bash
# Build locally first
cd examples/time-server-js/
npm run build

# Load from file
wassette component load file://$(pwd)/time.wasm

# Use via AI agent
```

## Testing Examples

### Interactive Testing with MCP Inspector

```bash
# Start Wassette
wassette serve --sse

# In another terminal, start inspector
npx @modelcontextprotocol/inspector --cli http://127.0.0.1:9001/sse

# List available tools
tools/list

# Call a tool
tools/call <tool-name> <arguments>
```

### Unit Testing

Each example may include language-specific tests:

**JavaScript**:
```bash
npm test
```

**Rust**:
```bash
cargo test
```

**Python**:
```bash
pytest
```

## Community Examples

Beyond the official examples, the community has built amazing components:

### QR Code Generator
- **Author**: @attackordie
- **Repository**: https://github.com/attackordie/qr-code-webassembly
- **Description**: Generate QR codes from text using WebAssembly
- **Language**: Rust

Want to add your example? [Open a pull request](https://github.com/microsoft/wassette/pulls)!

## Example Templates

### Quick Start Templates

Start a new component using these templates:

**JavaScript Template**:
```bash
# Coming soon: Component template generator
# wassette new --language javascript my-component
```

**Python Template**:
```bash
# Coming soon
# wassette new --language python my-component
```

**Rust Template**:
```bash
cargo component new my-component
cd my-component
# Edit wit/world.wit and src/lib.rs
cargo component build
```

## Best Practices

When studying or creating examples:

1. **Start Simple**: Begin with time-server-js for basics
2. **Understand Permissions**: Study how each example requests permissions
3. **Error Handling**: Note how examples handle errors with result types
4. **WIT Design**: Study interface designs for different use cases
5. **Security**: Review permission requirements before loading

## Contributing Examples

We welcome new examples! Guidelines:

1. **Clear Purpose**: Example should demonstrate a specific concept
2. **Complete Documentation**: Include README with usage instructions
3. **WIT Definition**: Provide well-documented WIT files
4. **Build Instructions**: Include clear build steps
5. **Permissions**: Document required permissions
6. **Tests**: Include basic tests if applicable

See [CONTRIBUTING.md](https://github.com/microsoft/wassette/blob/main/CONTRIBUTING.md) for details.

## See Also

- [Getting Started](./getting-started.md) - First-time Wassette setup
- [JavaScript Development](./development/javascript.md) - Building JS components
- [Python Development](./development/python.md) - Building Python components
- [Rust Development](./development/rust.md) - Building Rust components
- [Go Development](./development/go.md) - Building Go components
- [Permission System](./design/permission-system.md) - Understanding security
- [CLI Reference](./cli.md) - Command-line tools
