# Getting Started with Wassette

This tutorial will guide you through installing Wassette, loading your first WebAssembly component, and understanding the basics of the security model.

## Prerequisites

- A terminal/command line interface
- An MCP-compatible AI agent (VS Code with Copilot, Cursor, Claude Desktop, etc.)
- Basic familiarity with command line operations

## Step 1: Installation

Choose the installation method for your platform:

### Linux and macOS

Use the installation script for the quickest setup:

```bash
curl -fsSL https://raw.githubusercontent.com/microsoft/wassette/main/install.sh | bash
```

This script will:
- Detect your platform automatically
- Download the latest `wassette` binary
- Install it to your `$PATH`
- Verify the installation

### macOS with Homebrew

If you prefer package managers:

```bash
brew tap microsoft/wassette
brew install wassette
```

See the [Homebrew installation guide](./homebrew.md) for more details.

### Windows with WinGet

For Windows users:

```powershell
winget install Microsoft.Wassette
```

See the [WinGet installation guide](./winget.md) for more details.

### Nix

For reproducible environments:

```bash
nix profile install github:microsoft/wassette
```

See the [Nix installation guide](./nix.md) for more details.

### Verify Installation

After installation, verify that Wassette is available:

```bash
wassette --version
```

You should see output showing the installed version.

## Step 2: Configure Your AI Agent

Wassette works as an MCP server that connects to AI agents. You need to register Wassette with your agent of choice.

### Visual Studio Code

For VS Code users with GitHub Copilot, click the installation badge:

[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Server-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect?url=vscode:mcp/install?%7B%22name%22%3A%22wassette%22%2C%22gallery%22%3Afalse%2C%22command%22%3A%22wassette%22%2C%22args%22%3A%5B%22serve%22%2C%22--stdio%22%5D%7D)

Or use the command line:

```bash
code --add-mcp '{"name":"Wassette","command":"wassette","args":["serve","--stdio"]}'
```

### Other Agents

For detailed setup instructions for Cursor, Claude Desktop, Gemini CLI, and other agents, see the [MCP Clients Setup Guide](./mcp-clients.md).

## Step 3: Start Wassette

Start the Wassette MCP server. The method depends on your agent configuration:

### For Direct Testing (HTTP Mode)

To test Wassette independently:

```bash
wassette serve --sse
```

This starts Wassette with Server-Sent Events transport on `http://127.0.0.1:9001/sse`.

### For Agent Integration (Stdio Mode)

Most AI agents will automatically start Wassette when needed, using stdio transport. If you're manually starting it:

```bash
wassette serve --stdio
```

## Step 4: Load Your First Component

Now that Wassette is running and connected to your AI agent, let's load a simple component to teach your agent how to tell time.

### Using Your AI Agent

In your AI agent's chat interface, type:

```text
Please load the time component from oci://ghcr.io/yoshuawuyts/time:latest
```

The agent will:
1. Call Wassette's `load-component` tool
2. Download the component from the OCI registry
3. Load it into the Wassette runtime
4. Make the component's functions available as new tools

### What Just Happened?

- **OCI Registry**: WebAssembly components can be stored in OCI (Open Container Initiative) registries, just like container images
- **Dynamic Loading**: Wassette loaded the component at runtime without requiring a restart
- **Security Sandbox**: The component runs in a secure WebAssembly sandbox with no access to your system by default

## Step 5: Use the Component

Now that the time component is loaded, you can ask your agent to use it:

```text
What is the current time?
```

Your agent will:
1. Recognize it needs time information
2. Call the `get-current-time` function from the time component
3. Return the result to you

Example response:
```
The current time is January 31, 2025 at 10:30 AM UTC
```

## Step 6: Explore Available Components

Wassette includes a component registry with pre-built components you can discover:

```text
What components are available to load?
```

Your agent will call the `search-components` tool and show you a list of available components, such as:

- **Weather Server**: Fetch weather information for locations
- **Time Server**: Get current time and date
- **File System Tools**: Read and write files with controlled permissions
- **HTTP Client**: Make HTTP requests to specified domains

## Step 7: Understanding Security

One of Wassette's key features is its security model. Let's explore it.

### Viewing Permissions

Check what permissions a component has:

```text
Show me the policy for the time component
```

Your agent will call `get-policy` and show you the component's permissions. Initially, most components have minimal or no permissions.

### Granting Permissions

Components need explicit permissions to access resources. For example, if you load a weather component that needs network access:

```text
Grant the weather component permission to access api.openweathermap.org
```

This grants network access to that specific domain only. The component cannot access other network resources.

### Permission Types

Wassette supports several permission types:

- **Storage**: File system access (read/write to specific paths)
- **Network**: Network access (specific hosts only)
- **Environment Variables**: Access to specific environment variables
- **Memory**: Resource limits (maximum memory usage)

## Step 8: Load a Local Component

If you're developing your own components, you can load them from local files:

```text
Please load the component from file:///path/to/my-component.wasm
```

This is useful for:
- Testing components during development
- Using custom internal components
- Iterating quickly without publishing to a registry

## Next Steps

Now that you understand the basics, you can:

### Learn to Build Components

- [JavaScript/TypeScript Components](./development/javascript.md)
- [Python Components](./development/python.md)
- [Rust Components](./development/rust.md)
- [Go Components](./development/go.md)

### Explore Advanced Features

- [CLI Reference](./cli.md) - Direct command-line management
- [Permission System](./design/permission-system.md) - Deep dive into security
- [Architecture Overview](./design/architecture.md) - How Wassette works
- [Secret Management](./secrets.md) - Managing sensitive data

### Try Example Components

Browse the [examples directory](https://github.com/microsoft/wassette/tree/main/examples) for working components in multiple languages.

### Get Help

- Join the [Discord community](https://discord.gg/microsoft-open-source) in the `#wassette` channel
- Read the [FAQ](./faq.md) for common questions
- Check the [Troubleshooting Guide](./troubleshooting.md) for solutions to common issues
- Open an [issue on GitHub](https://github.com/microsoft/wassette/issues) for bugs or feature requests

## Common First-Time Issues

### Component Won't Load

**Problem**: Error loading a component from OCI registry

**Solutions**:
- Check your internet connection
- Verify the OCI path is correct
- Ensure the registry is accessible (not behind a firewall)
- Try the `--verbose` flag for detailed error messages

### Agent Can't Find Wassette

**Problem**: Agent reports "Wassette not found" or similar

**Solutions**:
- Verify `wassette` is in your PATH: `which wassette`
- Restart your AI agent after installation
- Check the agent's MCP configuration file
- Try running `wassette serve --stdio` manually to test

### Permission Denied Errors

**Problem**: Component fails with "permission denied" errors

**Solutions**:
- Check component permissions with `get-policy`
- Grant necessary permissions explicitly
- Remember: Wassette uses deny-by-default security
- Review the component's documentation for required permissions

## Summary

Congratulations! You've learned to:

✓ Install Wassette on your system  
✓ Configure it with your AI agent  
✓ Load WebAssembly components from registries  
✓ Use components through your agent  
✓ Understand Wassette's security model  
✓ Manage permissions for components  

Wassette provides a secure, flexible way to extend AI agents with WebAssembly components. The combination of sandboxed execution and fine-grained permissions makes it safe to run untrusted code while maintaining full control over system access.

Happy building! 🚀
