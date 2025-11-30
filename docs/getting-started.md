# Getting Started

This guide will walk you through setting up and using Wassette for the first time. By the end of this tutorial, you'll have Wassette running and be able to load and use WebAssembly components as tools for your AI agent.

## Prerequisites

Before you begin, ensure you have:

1. **Wassette installed** - See the [Installation guide](./installation.md) if you haven't installed it yet
2. **An MCP client** - Such as Claude Desktop, Cursor, VS Code with GitHub Copilot, or another MCP-compatible client

## Step 1: Verify Installation

First, confirm that Wassette is properly installed:

```bash
wassette --version
```

You should see output showing the version number, such as `wassette 0.1.0`.

## Step 2: Configure Your MCP Client

Wassette needs to be registered with your MCP client so the AI agent can communicate with it. The setup varies by client:

### VS Code (Recommended)

The quickest way to add Wassette to VS Code is using the install badges:

[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Server-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect?url=vscode:mcp/install?%7B%22name%22%3A%22wassette%22%2C%22gallery%22%3Afalse%2C%22command%22%3A%22wassette%22%2C%22args%22%3A%5B%22serve%22%2C%22--stdio%22%5D%7D)

Or from the command line:

**bash/zsh:**
```bash
code --add-mcp '{"name":"Wassette","command":"wassette","args":["serve","--stdio"]}'
```

**PowerShell:**
```powershell
code --% --add-mcp "{\"name\":\"wassette\",\"command\":\"wassette\",\"args\":[\"serve\",\"--stdio\"]}"
```

### Claude Desktop

Edit your Claude Desktop configuration file:

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`

**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

Add the following configuration:

```json
{
  "mcpServers": {
    "wassette": {
      "command": "wassette",
      "args": ["serve", "--stdio"]
    }
  }
}
```

### Cursor

Add to your Cursor configuration (`~/.cursor/config.json` or similar):

```json
{
  "mcpServers": {
    "wassette": {
      "command": "wassette",
      "args": ["serve", "--stdio"]
    }
  }
}
```

For more detailed setup instructions for different clients, see the [MCP Clients guide](./mcp-clients.md).

## Step 3: Start Your MCP Client

Restart your MCP client (VS Code, Claude Desktop, etc.) to load the Wassette server. Once restarted, Wassette will be running in the background, ready to accept commands from your AI agent.

## Step 4: Load Your First Component

Now let's teach your AI agent a new capability by loading a WebAssembly component. We'll start with a simple time component.

In your MCP client's chat interface, send this message:

```text
Please load the time component from oci://ghcr.io/yoshuawuyts/time:latest
```

The AI agent will use Wassette's `load-component` tool to fetch and load the component. You should see a confirmation that the component was loaded successfully, along with a list of new tools that are now available.

## Step 5: Use the Loaded Component

Now that the time component is loaded, you can use its functionality. Ask your agent:

```text
What is the current time?
```

The agent will call the appropriate tool from the time component and respond with the current date and time, for example:

```output
The current time is January 15, 2025 at 2:30 PM UTC
```

Congratulations! You've just run your first WebAssembly component through Wassette! 🎉

## Step 6: Explore Available Components

Wassette maintains a registry of known components. To see what's available, ask your agent:

```text
What components can I load with Wassette?
```

The agent will use the `search-components` tool to list available components from the registry. You'll see components for various purposes like weather data, file operations, code execution, and more.

## Step 7: Manage Component Permissions

Wassette uses a capability-based security model. Let's explore this by loading a component that needs permissions:

```text
Please load the filesystem component from oci://ghcr.io/microsoft/filesystem-rs:latest
```

Once loaded, the component has no permissions by default. To grant it access to a specific directory:

```text
Grant the filesystem component read and write access to /tmp/test
```

The agent will use the `grant-storage-permission` tool to add this capability. Now you can ask:

```text
List the files in /tmp/test
```

The component can now access that specific directory while remaining isolated from the rest of your filesystem.

## Common Operations

### Listing Loaded Components

```text
Show me all currently loaded components
```

This uses the `list-components` tool to display active components and their tools.

### Unloading Components

```text
Unload the time component
```

This removes the component and frees its resources.

### Viewing Component Policies

```text
Show me the policy for the filesystem component
```

This displays the security policy and permissions for a specific component.

### Revoking Permissions

```text
Revoke storage access from the filesystem component for /tmp/test
```

This removes previously granted permissions.

## Understanding Component Locations

Components can be loaded from different sources:

1. **OCI Registries** (recommended): `oci://ghcr.io/owner/component:tag`
   - Most common and convenient
   - Supports versioning with tags
   - Example: `oci://ghcr.io/yoshuawuyts/time:latest`

2. **Local Files**: `file:///absolute/path/to/component.wasm`
   - Useful for development
   - Must be absolute paths
   - Example: `file:///home/user/my-component.wasm`

## What's Next?

Now that you've learned the basics, you can:

1. **Explore Examples** - Check out the [Examples guide](./examples.md) for more component use cases
2. **Build Your Own** - Learn to create components in [Rust](./development/rust.md), [JavaScript](./development/javascript.md), [Python](./development/python.md), or [Go](./development/go.md)
3. **Understand Security** - Read about the [Permission System](./design/permission-system.md) for fine-grained access control
4. **Use the CLI** - Learn about direct command-line management in the [CLI guide](./cli.md)

## Troubleshooting

### Component Won't Load

If a component fails to load:

1. **Check the URI**: Ensure the OCI registry path or file path is correct
2. **Network Access**: Verify you can reach the OCI registry (if using `oci://`)
3. **Component Format**: Ensure the file is a valid WebAssembly component (not a module)
4. **Check Logs**: Look at the MCP client logs for detailed error messages

### Permission Denied Errors

If you get permission errors when using a component:

1. **Grant Required Permissions**: Use `grant-*-permission` tools to add necessary capabilities
2. **Check Policy**: Use `get-policy` to see what permissions are currently granted
3. **Be Specific**: Permissions are path/host-specific; grant exactly what's needed

### Agent Can't Find Tools

If the AI agent doesn't recognize Wassette tools:

1. **Restart Client**: Make sure you restarted your MCP client after configuration
2. **Check Configuration**: Verify the configuration file syntax is correct
3. **Test Connection**: Ask the agent to list available tools

### Component Not Working as Expected

1. **Check Component Documentation**: Each component should document its tools and requirements
2. **Verify Inputs**: Ensure you're providing the correct parameters
3. **Check Permissions**: Some operations require specific granted permissions
4. **Review Examples**: Look at the [Examples](./examples.md) for working use cases

## Getting Help

If you need assistance:

- **Documentation**: Browse the complete [documentation](https://microsoft.github.io/wassette)
- **FAQ**: Check the [FAQ](./faq.md) for common questions
- **Examples**: Review the [Examples](./examples.md) for working code
- **Discord**: Join the [Discord community](https://discord.gg/microsoft-open-source) (#wassette channel)
- **GitHub Issues**: [Report bugs or request features](https://github.com/microsoft/wassette/issues)

Happy building! 🚀
