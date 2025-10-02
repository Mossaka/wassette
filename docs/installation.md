# Installation

This guide provides comprehensive instructions for installing Wassette on different platforms. Choose the installation method that best fits your needs and operating system.

## Quick Install (Linux & macOS)

For Linux (including Windows Subsystem for Linux) and macOS, you can install Wassette using our install script:

```bash
curl -fsSL https://raw.githubusercontent.com/microsoft/wassette/main/install.sh | bash
```

This script will:
- Automatically detect your platform (Linux or macOS)
- Download the latest release for your architecture
- Install the `wassette` binary to your `$PATH`
- Verify the installation

## Installation Methods by Platform

### macOS

#### Homebrew (Recommended)

The easiest way to install Wassette on macOS is using Homebrew:

```bash
brew install microsoft/wassette/wassette
```

For more details, see the [Homebrew installation guide](./homebrew.md).

#### Manual Installation

Download the latest release for macOS:

```bash
# For Apple Silicon (M1/M2/M3)
curl -LO https://github.com/microsoft/wassette/releases/latest/download/wassette-aarch64-apple-darwin.tar.gz
tar xzf wassette-aarch64-apple-darwin.tar.gz
sudo mv wassette /usr/local/bin/

# For Intel Macs
curl -LO https://github.com/microsoft/wassette/releases/latest/download/wassette-x86_64-apple-darwin.tar.gz
tar xzf wassette-x86_64-apple-darwin.tar.gz
sudo mv wassette /usr/local/bin/
```

### Linux

#### Quick Install Script (Recommended)

```bash
curl -fsSL https://raw.githubusercontent.com/microsoft/wassette/main/install.sh | bash
```

#### Homebrew on Linux

```bash
brew install microsoft/wassette/wassette
```

For more details, see the [Homebrew installation guide](./homebrew.md).

#### Nix

For users who prefer Nix for reproducible environments:

```bash
nix profile install github:microsoft/wassette
```

For more details including development shells, see the [Nix installation guide](./nix.md).

#### Manual Installation

Download the appropriate release for your architecture:

```bash
# For x86_64 Linux
curl -LO https://github.com/microsoft/wassette/releases/latest/download/wassette-x86_64-unknown-linux-gnu.tar.gz
tar xzf wassette-x86_64-unknown-linux-gnu.tar.gz
sudo mv wassette /usr/local/bin/

# For ARM64 Linux
curl -LO https://github.com/microsoft/wassette/releases/latest/download/wassette-aarch64-unknown-linux-gnu.tar.gz
tar xzf wassette-aarch64-unknown-linux-gnu.tar.gz
sudo mv wassette /usr/local/bin/
```

### Windows

#### WinGet (Recommended)

The easiest way to install Wassette on Windows is using WinGet:

```powershell
winget install Microsoft.Wassette
```

For more details, see the [WinGet installation guide](./winget.md).

#### Manual Installation

1. Download the latest Windows release from the [GitHub Releases page](https://github.com/microsoft/wassette/releases)
2. Extract the archive
3. Add the `wassette.exe` location to your PATH environment variable

### Windows Subsystem for Linux (WSL)

If you're using WSL, follow the Linux installation instructions above.

## Building from Source

If you prefer to build from source or need the latest development version:

### Prerequisites

- Rust 1.75.0 or later
- Git

### Build Steps

```bash
# Clone the repository
git clone https://github.com/microsoft/wassette.git
cd wassette

# Build the release binary
cargo build --release

# The binary will be at target/release/wassette
# Copy it to your PATH
sudo cp target/release/wassette /usr/local/bin/
```

For more detailed development setup instructions, see the [Development Setup guide](./development.md).

## Verification

After installation, verify that Wassette is installed correctly:

```bash
# Check version
wassette --version

# Display help
wassette --help

# Test basic functionality
wassette serve --help
```

You should see output showing the version number and available commands.

## Next Steps

Once Wassette is installed, you can:

1. **Configure your MCP client** - See the [MCP Clients guide](./mcp-clients.md) for setup instructions for Cursor, Claude Code, VS Code, and other clients
2. **Try the quick start** - Follow the [Getting Started guide](./getting-started.md) to load your first component
3. **Explore examples** - Check out the [Examples](./examples.md) to see what you can build

## Troubleshooting

### Command Not Found

If you get a "command not found" error after installation:

1. **Check your PATH**: Ensure the installation directory is in your PATH
   ```bash
   echo $PATH
   ```

2. **Reload your shell**: Close and reopen your terminal, or run:
   ```bash
   source ~/.bashrc  # or ~/.zshrc for zsh
   ```

3. **Manual PATH setup**: Add to your shell configuration file:
   ```bash
   export PATH="$PATH:/usr/local/bin"
   ```

### Permission Denied

If you get permission errors:

- On Linux/macOS: Use `sudo` for system-wide installation
- On Windows: Run PowerShell as Administrator
- Alternative: Install to a user directory that doesn't require elevated privileges

### Platform-Specific Issues

- **macOS**: If you see "unverified developer" warnings, go to System Preferences → Security & Privacy and allow the application
- **Linux**: Ensure you have the required system libraries (glibc 2.27 or later)
- **Windows**: Make sure you have the Visual C++ Redistributable installed

## Updating

To update to the latest version:

**Homebrew:**
```bash
brew upgrade wassette
```

**WinGet:**
```powershell
winget upgrade Microsoft.Wassette
```

**Nix:**
```bash
nix profile upgrade wassette
```

**Manual installation:**
Download and install the latest release following the same steps as initial installation.

## Uninstalling

**Homebrew:**
```bash
brew uninstall wassette
```

**WinGet:**
```powershell
winget uninstall Microsoft.Wassette
```

**Nix:**
```bash
nix profile remove wassette
```

**Manual installation:**
```bash
sudo rm /usr/local/bin/wassette
```

## Support

If you encounter issues during installation:

- Check the [FAQ](./faq.md) for common questions
- Search [existing issues](https://github.com/microsoft/wassette/issues) on GitHub
- Join the [Discord community](https://discord.gg/microsoft-open-source) (#wassette channel)
- [Open a new issue](https://github.com/microsoft/wassette/issues/new) if your problem isn't already reported
