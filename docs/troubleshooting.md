# Troubleshooting Guide

This guide helps you diagnose and resolve common issues when working with Wassette.

## Table of Contents

- [Installation Issues](#installation-issues)
- [Component Loading Problems](#component-loading-problems)
- [Permission Errors](#permission-errors)
- [Network Issues](#network-issues)
- [Performance Problems](#performance-problems)
- [MCP Client Integration](#mcp-client-integration)
- [Debugging Tips](#debugging-tips)

## Installation Issues

### Wassette Command Not Found

**Symptom**: Running `wassette` in terminal returns "command not found"

**Possible Causes**:
1. Wassette is not installed
2. Wassette is not in your PATH
3. Terminal session hasn't reloaded PATH

**Solutions**:

```bash
# Verify installation
which wassette

# If not found, check common installation locations
ls -la ~/.local/bin/wassette     # Linux/macOS local install
ls -la /usr/local/bin/wassette   # System-wide install

# Add to PATH if needed (add to ~/.bashrc or ~/.zshrc)
export PATH="$HOME/.local/bin:$PATH"

# Reload terminal
source ~/.bashrc  # or source ~/.zshrc
```

### Permission Denied on Installation

**Symptom**: Installation script fails with "Permission denied"

**Solutions**:

```bash
# Use local installation (no sudo needed)
curl -fsSL https://raw.githubusercontent.com/microsoft/wassette/main/install.sh | bash

# Or install to custom directory
curl -fsSL https://raw.githubusercontent.com/microsoft/wassette/main/install.sh | INSTALL_DIR=~/bin bash

# Make sure install directory has correct permissions
chmod +x ~/bin/wassette
```

### Version Mismatch

**Symptom**: Wassette reports unexpected version

**Solutions**:

```bash
# Check current version
wassette --version

# Remove old installation
rm $(which wassette)

# Reinstall latest version
curl -fsSL https://raw.githubusercontent.com/microsoft/wassette/main/install.sh | bash

# Verify new version
wassette --version
```

## Component Loading Problems

### OCI Registry Connection Failed

**Symptom**: Error loading component from OCI registry

```
Error: failed to fetch component from oci://ghcr.io/example/component:latest
```

**Possible Causes**:
1. No internet connection
2. Registry is down or inaccessible
3. Component doesn't exist at that path
4. Authentication required but not provided

**Solutions**:

```bash
# Test internet connectivity
ping -c 3 ghcr.io

# Verify component path
# Check the repository for correct path

# Test with HTTP mode for better error messages
wassette serve --http
# Then check logs

# Enable debug logging
RUST_LOG=debug wassette serve --stdio

# Try pulling with docker to test registry access
docker pull ghcr.io/example/component:latest
```

### Invalid Component Format

**Symptom**: Component loads but fails validation

```
Error: invalid component format
```

**Possible Causes**:
1. File is not a valid WebAssembly Component
2. Component is a module, not a Component (different format)
3. Corrupted download

**Solutions**:

```bash
# Verify it's a Component (not just a module)
wasm-tools component wit ./component.wasm

# Validate the component
wasm-tools validate ./component.wasm

# Check component metadata
wasm-tools metadata show ./component.wasm

# Re-download if from OCI registry
rm ~/.local/share/wassette/components/component-id.wasm
# Then load again
```

### Component Already Loaded

**Symptom**: Attempting to load a component that's already loaded

```
Error: component with id 'example' is already loaded
```

**Solutions**:

```bash
# List loaded components
wassette component list

# Unload the existing component
wassette component unload example

# Load the new version
wassette component load oci://ghcr.io/example/component:latest

# Or reload by loading over the existing component
# Wassette will automatically reload if paths match
```

### Component File Not Found

**Symptom**: Local component file cannot be loaded

```
Error: file not found: /path/to/component.wasm
```

**Solutions**:

```bash
# Verify file exists
ls -la /path/to/component.wasm

# Use absolute path
wassette component load file:///absolute/path/to/component.wasm

# Check file permissions
chmod 644 /path/to/component.wasm

# Verify it's a Wasm file
file /path/to/component.wasm
# Should report: WebAssembly (wasm) binary module
```

## Permission Errors

### Access Denied: File System

**Symptom**: Component reports file access denied

```
Error: permission denied: fs:///workspace/file.txt
```

**Possible Causes**:
1. Component doesn't have storage permission
2. Path is outside allowed directories
3. Permission is read-only but write was attempted

**Solutions**:

```bash
# Check current permissions
wassette policy get component-id

# Grant storage permission
wassette permission grant storage component-id fs:///workspace/ --access read,write

# Grant read-only permission
wassette permission grant storage component-id fs:///workspace/ --access read

# For specific file
wassette permission grant storage component-id fs:///workspace/file.txt --access read,write

# Verify permission was granted
wassette policy get component-id
```

### Access Denied: Network

**Symptom**: Component cannot make network requests

```
Error: network access denied: api.example.com
```

**Possible Causes**:
1. Component doesn't have network permission
2. Domain is not in allowed list
3. Attempting HTTPS but HTTP allowed (or vice versa)

**Solutions**:

```bash
# Check current permissions
wassette policy get component-id

# Grant network permission
wassette permission grant network component-id api.example.com

# For different port
wassette permission grant network component-id api.example.com:8080

# Verify permission
wassette policy get component-id
```

### Environment Variable Access Denied

**Symptom**: Component cannot read environment variable

```
Error: environment variable 'API_KEY' not accessible
```

**Solutions**:

```bash
# Grant environment variable permission
wassette permission grant environment-variable component-id API_KEY

# Set the value via secrets
wassette secret set component-id API_KEY your-key-here

# Or via policy
cat > policy.yaml <<EOF
version: "1.0"
permissions:
  environment:
    allow:
      - key: API_KEY
        value: "your-key-here"
EOF

# Verify
wassette policy get component-id
wassette secret list component-id
```

### Too Many Permissions

**Symptom**: Component has overly broad permissions, security concern

**Solutions**:

```bash
# Review current permissions
wassette policy get component-id

# Reset all permissions
wassette permission reset component-id

# Grant only necessary permissions
wassette permission grant storage component-id fs:///workspace/data/ --access read
wassette permission grant network component-id api.example.com

# Verify minimal permissions
wassette policy get component-id
```

## Network Issues

### Connection Timeout

**Symptom**: Network requests from component time out

**Possible Causes**:
1. Target host is slow or unresponsive
2. Firewall blocking connection
3. DNS resolution failure
4. Component hitting resource limits

**Solutions**:

```bash
# Test connectivity outside Wassette
curl https://api.example.com

# Check DNS resolution
nslookup api.example.com

# Enable debug logging
RUST_LOG=debug wassette serve --stdio

# Increase memory limit if applicable
wassette permission grant memory component-id 1Gi

# Check network permissions
wassette policy get component-id
```

### SSL/TLS Errors

**Symptom**: HTTPS requests fail with certificate errors

```
Error: SSL certificate verification failed
```

**Solutions**:

```bash
# Update CA certificates (Linux)
sudo update-ca-certificates

# macOS
# System certificates usually up to date

# Check target host certificate
openssl s_client -connect api.example.com:443 -showcerts

# Temporarily test with HTTP (not for production)
# Update component to use http:// instead of https://
```

### Proxy Configuration

**Symptom**: Network requests fail behind corporate proxy

**Solutions**:

```bash
# Set proxy environment variables
export HTTP_PROXY=http://proxy.company.com:8080
export HTTPS_PROXY=http://proxy.company.com:8080
export NO_PROXY=localhost,127.0.0.1

# Start Wassette with proxy settings
wassette serve --stdio

# Or configure in component policy
cat > policy.yaml <<EOF
version: "1.0"
permissions:
  environment:
    allow:
      - key: HTTP_PROXY
        value: "http://proxy.company.com:8080"
      - key: HTTPS_PROXY
        value: "http://proxy.company.com:8080"
EOF
```

## Performance Problems

### Slow Component Loading

**Symptom**: Components take a long time to load

**Possible Causes**:
1. Large component file
2. Slow network connection
3. Heavy computation during initialization

**Solutions**:

```bash
# Check component size
ls -lh ~/.local/share/wassette/components/

# Use local cache
# After first download, components are cached locally

# Optimize component
# - Reduce dependencies
# - Remove debug symbols
# - Use release builds

# Pre-download components
wassette component load oci://ghcr.io/example/component:latest
# This caches for future use
```

### High Memory Usage

**Symptom**: Wassette or components consuming excessive memory

**Possible Causes**:
1. Memory leaks in component
2. Large data structures
3. No memory limits set

**Solutions**:

```bash
# Monitor memory usage
ps aux | grep wassette

# Set memory limits
wassette permission grant memory component-id 512Mi

# Use smaller memory limit
wassette permission grant memory component-id 256Mi

# Review component code for memory leaks
# Check for unbounded growth in collections

# Restart Wassette to clear memory
# (with --stdio, agent will restart automatically)
```

### Slow Tool Execution

**Symptom**: Component functions take longer than expected

**Possible Causes**:
1. Unoptimized component code
2. Excessive I/O operations
3. Debug build instead of release build
4. Resource constraints

**Solutions**:

```bash
# Rebuild component with optimizations
# Rust:
cargo build --release --target wasm32-wasip2

# JavaScript:
jco componentize --optimize main.js -o component.wasm

# Profile component execution
RUST_LOG=trace wassette serve --stdio
# Check logs for timing information

# Increase resource limits
wassette permission grant memory component-id 1Gi
```

## MCP Client Integration

### VS Code Copilot Can't Find Wassette

**Symptom**: VS Code reports Wassette MCP server unavailable

**Solutions**:

```bash
# Verify Wassette in PATH
which wassette

# Reinstall MCP server in VS Code
code --add-mcp '{"name":"Wassette","command":"wassette","args":["serve","--stdio"]}'

# Reload VS Code window
# Cmd/Ctrl + Shift + P -> "Developer: Reload Window"

# Check VS Code MCP settings
# Open settings and search for "MCP"

# Check Output panel in VS Code
# View -> Output -> Select "MCP: Wassette"
```

### Cursor Integration Issues

**Symptom**: Cursor IDE doesn't connect to Wassette

**Solutions**:

```bash
# Check Cursor MCP configuration
cat ~/.cursor/mcp_settings.json

# Should contain Wassette configuration:
# {
#   "wassette": {
#     "command": "wassette",
#     "args": ["serve", "--stdio"]
#   }
# }

# Restart Cursor completely
# Not just reload window

# Check Cursor logs
# Help -> Show Logs -> Extension Host
```

### Claude Desktop Connection

**Symptom**: Claude Desktop cannot communicate with Wassette

**Solutions**:

```bash
# Check Claude config file
# macOS:
cat ~/Library/Application\ Support/Claude/claude_desktop_config.json

# Linux:
cat ~/.config/claude/claude_desktop_config.json

# Should contain:
# {
#   "mcpServers": {
#     "wassette": {
#       "command": "wassette",
#       "args": ["serve", "--stdio"]
#     }
#   }
# }

# Restart Claude Desktop
# Quit completely and reopen

# Enable debug mode
# Add environment variable:
export MCP_DEBUG=1
# Then start Claude Desktop from terminal
```

## Debugging Tips

### Enable Debug Logging

Get detailed information about what Wassette is doing:

```bash
# Maximum verbosity
RUST_LOG=trace wassette serve --stdio

# Debug level (recommended)
RUST_LOG=debug wassette serve --stdio

# Specific module
RUST_LOG=wassette::mcp=debug wassette serve --stdio

# Multiple modules
RUST_LOG=wassette=debug,policy=trace wassette serve --stdio
```

### Use MCP Inspector

Test Wassette independently of your AI agent:

```bash
# Start Wassette in HTTP mode
wassette serve --sse

# In another terminal, use MCP Inspector
npx @modelcontextprotocol/inspector --cli http://127.0.0.1:9001/sse

# Available commands:
# - tools/list: List all available tools
# - tools/call: Call a specific tool
# - resources/list: List resources
```

### Check Component Logs

Components can output logs:

```bash
# JavaScript component console.log appears in Wassette logs
export RUST_LOG=debug
wassette serve --stdio
# Look for component output in logs

# Redirect logs to file
wassette serve --stdio 2> wassette.log
# Check wassette.log file
```

### Validate Component WIT

Ensure your component's interface is correct:

```bash
# Install wasm-tools
cargo install wasm-tools

# Extract WIT from component
wasm-tools component wit ./component.wasm

# Validate WIT syntax
wasm-tools component wit ./wit/world.wit

# Show component metadata
wasm-tools metadata show ./component.wasm
```

### Test Component Independently

Before loading into Wassette, test components:

```bash
# JavaScript components
node --experimental-wasm-components main.js

# Rust components
cargo test

# With wasmtime directly
wasmtime serve ./component.wasm
```

### Inspect Component Registry

Check cached components:

```bash
# List downloaded components
ls -la ~/.local/share/wassette/components/

# Check component size
du -sh ~/.local/share/wassette/components/*

# Remove cached component to force re-download
rm ~/.local/share/wassette/components/component-id.wasm

# Clear all cached components
rm -rf ~/.local/share/wassette/components/
```

## Getting Help

If you've tried the solutions above and still have issues:

### Gather Information

Before asking for help, collect:

1. **Version information**:
   ```bash
   wassette --version
   ```

2. **Debug logs**:
   ```bash
   RUST_LOG=debug wassette serve --stdio 2> wassette-debug.log
   ```

3. **System information**:
   ```bash
   uname -a  # OS and architecture
   ```

4. **Component information** (if applicable):
   ```bash
   wasm-tools component wit ./component.wasm
   ```

### Community Support

- **Discord**: Join `#wassette` channel on [Microsoft Open Source Discord](https://discord.gg/microsoft-open-source)
- **GitHub Issues**: Open an issue at https://github.com/microsoft/wassette/issues
- **GitHub Discussions**: Ask questions at https://github.com/microsoft/wassette/discussions

### Issue Template

When reporting issues, include:

```markdown
**Environment:**
- OS: [Linux/macOS/Windows]
- Wassette version: [output of `wassette --version`]
- MCP Client: [VS Code/Cursor/Claude Desktop/etc.]

**Problem Description:**
[Clear description of the issue]

**Steps to Reproduce:**
1. 
2. 
3. 

**Expected Behavior:**
[What you expected to happen]

**Actual Behavior:**
[What actually happened]

**Logs:**
```
[Paste debug logs here]
```

**Additional Context:**
[Any other relevant information]
```

## See Also

- [FAQ](./faq.md) - Frequently asked questions
- [Getting Started](./getting-started.md) - Basic usage guide
- [CLI Reference](./cli.md) - Complete CLI documentation
- [Permission System](./design/permission-system.md) - Security model details
- [Examples](https://github.com/microsoft/wassette/tree/main/examples) - Working examples
