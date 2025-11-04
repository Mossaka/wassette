# Secrets Management

Wassette provides a secure, per-component secrets management system that allows you to store sensitive configuration data (like API keys, tokens, and passwords) separately from your component code and policies.

## Overview

The secrets management system offers:

- **Per-Component Isolation**: Each component has its own separate secrets storage
- **Secure Storage**: Secrets are stored in OS-appropriate directories with restricted permissions (0700/user-only on Unix-like systems)
- **Persistence**: Secrets persist across runs without requiring server restart
- **Easy Management**: Simple CLI commands for listing, setting, and deleting secrets
- **Integration**: Seamlessly integrates with the environment variable precedence system
- **Audit Trail**: YAML format for easy editing and auditing

## Storage Location

Secrets are stored in platform-specific directories:

- **Linux/BSD**: `$XDG_DATA_HOME/wassette/secrets/` (typically `~/.local/share/wassette/secrets/`)
- **macOS**: `~/Library/Application Support/wassette/secrets/`
- **Windows**: `%APPDATA%\wassette\secrets\`

Each component's secrets are stored in a separate YAML file named `{component-id}.secrets.yaml`.

## Environment Variable Precedence

When a component requests an environment variable, Wassette follows this precedence order (highest to lowest):

1. **Policy-defined variables**: Environment variables explicitly allowed in the component's policy file
2. **Secrets**: Values from the component's secrets file
3. **Inherited environment**: Variables inherited from the parent process

This ensures that policy-defined variables always take precedence, maintaining security boundaries.

## CLI Commands

### List Secrets

View all secret keys for a component (without showing values by default):

```bash
# List secret keys only
wassette secret list <component-id>

# List with values (prompts for confirmation)
wassette secret list <component-id> --show-values

# Skip confirmation prompt
wassette secret list <component-id> --show-values --yes

# Use custom plugin directory
wassette secret list <component-id> --plugin-dir /custom/components

# Output in different format
wassette secret list <component-id> -o json
wassette secret list <component-id> -o yaml
wassette secret list <component-id> -o table
```

**Example output (keys only):**
```json
{
  "status": "success",
  "component_id": "time-component",
  "secret_keys": ["API_KEY", "DATABASE_URL", "SECRET_TOKEN"]
}
```

**Example output (with values):**
```json
{
  "status": "success",
  "component_id": "time-component",
  "secrets": {
    "API_KEY": "sk_live_abc123...",
    "DATABASE_URL": "postgresql://user:pass@localhost/db",
    "SECRET_TOKEN": "token_xyz789..."
  }
}
```

### Set Secrets

Add or update secrets for a component:

```bash
# Set a single secret
wassette secret set <component-id> API_KEY=sk_live_abc123

# Set multiple secrets at once
wassette secret set <component-id> API_KEY=sk_live_abc123 DATABASE_URL=postgresql://localhost/db

# Use custom plugin directory
wassette secret set <component-id> API_KEY=sk_live_abc123 --plugin-dir /custom/components
```

**Notes:**
- If a secret with the same key already exists, it will be updated
- Secrets are written immediately to disk
- No server restart is required
- The secrets file is created with restricted permissions (0600 on Unix-like systems)

### Delete Secrets

Remove specific secrets from a component:

```bash
# Delete a single secret
wassette secret delete <component-id> API_KEY

# Delete multiple secrets
wassette secret delete <component-id> API_KEY DATABASE_URL

# Use custom plugin directory
wassette secret delete <component-id> API_KEY --plugin-dir /custom/components
```

**Notes:**
- Only the specified keys are removed
- If a key doesn't exist, it's silently ignored
- The secrets file is preserved (even if empty)

## Security Considerations

### File Permissions

On Unix-like systems (Linux, macOS, BSD), secrets files are created with the following permissions:

- **Secrets directory**: `0700` (rwx------, only owner can access)
- **Secrets files**: `0600` (rw-------, only owner can read/write)

On Windows, the files are created with default NTFS permissions for the current user.

### Access Control

- Secrets are **per-component**: One component cannot access another component's secrets
- Secrets are **user-scoped**: Each user has their own separate secrets storage
- The secrets manager uses **lazy loading** with mtime-based cache invalidation for performance while ensuring secrets are always up-to-date

### Best Practices

1. **Rotate secrets regularly**: Update API keys and tokens periodically
2. **Use minimal permissions**: Only grant access to secrets that are absolutely necessary
3. **Never commit secrets**: Keep secrets out of version control
4. **Audit access**: Periodically review which secrets are configured for each component
5. **Use policy constraints**: Combine secrets with policy-defined environment variable restrictions

## File Format

Secrets are stored in a simple YAML format with flat String→String mappings:

```yaml
# Example: time-component.secrets.yaml
API_KEY: "sk_live_abc123..."
DATABASE_URL: "postgresql://user:pass@localhost/db"
SECRET_TOKEN: "token_xyz789..."
DEBUG_MODE: "false"
```

**Format characteristics:**
- Simple key-value pairs
- All values are strings
- Easy to edit manually (if needed)
- Easy to audit and review
- Supports comments for documentation

## Integration with Components

### Accessing Secrets in Components

Secrets are automatically injected as environment variables when a component is instantiated. Components can access them using standard environment variable APIs:

**Rust example:**
```rust
use std::env;

fn main() {
    // Access secret from environment
    let api_key = env::var("API_KEY")
        .expect("API_KEY not found in environment");
    
    println!("Using API key: {}", &api_key[..8]); // Print first 8 chars only
}
```

**JavaScript/TypeScript example:**
```javascript
// Access secret from environment
const apiKey = process.env.API_KEY;

if (!apiKey) {
    throw new Error('API_KEY not found in environment');
}

console.log(`Using API key: ${apiKey.substring(0, 8)}...`);
```

**Python example:**
```python
import os

# Access secret from environment
api_key = os.environ.get('API_KEY')

if not api_key:
    raise ValueError('API_KEY not found in environment')

print(f'Using API key: {api_key[:8]}...')
```

**Go example:**
```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // Access secret from environment
    apiKey := os.Getenv("API_KEY")
    
    if apiKey == "" {
        panic("API_KEY not found in environment")
    }
    
    fmt.Printf("Using API key: %s...\n", apiKey[:8])
}
```

### Requesting Environment Variables in Policy

To use secrets, components must first declare their environment variable requirements in their policy file:

```yaml
# component-policy.yaml
version: "1.0"
description: "Component with secret access"
permissions:
  environment:
    allow:
      - key: "API_KEY"
      - key: "DATABASE_URL"
      - key: "SECRET_TOKEN"
```

If a component requests an environment variable that's not allowed in its policy, access will be denied for security reasons.

## Workflow Examples

### Example 1: Setting Up a New Component with Secrets

```bash
# 1. Load the component
wassette component load oci://ghcr.io/example/api-client:latest

# 2. Note the component ID from the output
# component_id: "api-client-abc123"

# 3. Set required secrets
wassette secret set api-client-abc123 \
    API_KEY=sk_live_abc123 \
    API_ENDPOINT=https://api.example.com

# 4. Verify secrets were set
wassette secret list api-client-abc123

# 5. Start using the component (secrets will be available)
```

### Example 2: Rotating an API Key

```bash
# 1. List current secrets (to verify which key to rotate)
wassette secret list weather-component

# 2. Update the API key
wassette secret set weather-component API_KEY=sk_live_new_key_xyz

# 3. Verify update
wassette secret list weather-component --show-values --yes
```

### Example 3: Cleaning Up Secrets

```bash
# 1. Remove specific secrets
wassette secret delete old-component API_KEY DATABASE_URL

# 2. Verify deletion
wassette secret list old-component

# 3. Unload the component if no longer needed
wassette component unload old-component
```

### Example 4: Migrating Secrets Between Environments

```bash
# Export secrets from dev environment
wassette secret list my-component --show-values --yes -o yaml > secrets-backup.yaml

# On production environment, set secrets from backup
# (Parse YAML and set each secret individually)
wassette secret set my-component \
    API_KEY=<value-from-backup> \
    DATABASE_URL=<value-from-backup>
```

## Performance Considerations

The secrets management system is designed for high performance:

- **Lazy Loading**: Secrets files are only loaded when first accessed
- **Caching**: Loaded secrets are cached in memory with mtime-based invalidation
- **Fast Lookup**: In-memory HashMap for O(1) secret access
- **Minimal Overhead**: No performance impact if a component doesn't use secrets

## Troubleshooting

### Secrets Not Available to Component

**Problem**: Component can't access environment variables even after setting secrets.

**Solutions**:
1. Verify the environment variable is allowed in the component's policy:
   ```bash
   wassette policy get <component-id>
   ```
2. Check if the secret was actually set:
   ```bash
   wassette secret list <component-id> --show-values --yes
   ```
3. Ensure the secret key name matches exactly (case-sensitive)
4. Reload the component if secrets were added after loading

### Permission Denied Errors

**Problem**: Cannot read or write secrets files.

**Solutions**:
1. Check directory permissions:
   ```bash
   ls -la $(dirname $(wassette secret list <component-id> --plugin-dir . 2>&1 | grep -o '/.*secrets'))
   ```
2. Ensure you have write access to the secrets directory
3. On Unix-like systems, verify file ownership:
   ```bash
   stat $(dirname $(wassette secret list <component-id> --plugin-dir . 2>&1 | grep -o '/.*secrets'))
   ```

### Secrets File Corruption

**Problem**: YAML parsing errors when reading secrets.

**Solutions**:
1. Manually inspect the secrets file for syntax errors
2. Common issues:
   - Unquoted values with special characters
   - Incorrect indentation
   - Missing colons after keys
3. Fix the YAML syntax or delete and recreate the file:
   ```bash
   rm ~/.local/share/wassette/secrets/<component-id>.secrets.yaml
   wassette secret set <component-id> KEY=value
   ```

## Limitations

Current limitations of the secrets management system:

- **No encryption at rest**: Secrets are stored in plain text YAML files (rely on OS file permissions)
- **No secret rotation automation**: Manual rotation required
- **No secret versioning**: Previous secret values are not preserved
- **No audit logging**: No built-in tracking of secret access or modifications
- **No secret sharing**: Secrets cannot be shared between components
- **String values only**: All secret values are stored as strings

## Future Enhancements

Potential future improvements to the secrets management system:

- **Encryption at rest**: Encrypt secrets files using OS keychain/keyring
- **Secret rotation**: Automated rotation with version history
- **Audit logging**: Track when secrets are accessed or modified
- **Integration with secret managers**: Support for HashiCorp Vault, AWS Secrets Manager, etc.
- **Secret templates**: Define common secret patterns for component types
- **Bulk operations**: Import/export secrets in batch

## Related Documentation

- [CLI Reference](./cli.md) - Complete CLI command documentation
- [Permission System](./design/permission-system.md) - Understanding the permission model
- [Policy Files](./design/permission-system.md#policy-file-format) - Policy configuration format

---

*For questions, issues, or contributions related to secrets management, please see the [main repository](https://github.com/microsoft/wassette).*
