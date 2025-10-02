# Secret Management

Wassette provides a simple per-component secret management system that allows you to securely store and manage sensitive configuration data such as API keys, tokens, and credentials for WebAssembly components.

## Overview

The secret management system provides:

- **Per-Component Isolation**: Each component has its own separate secrets storage
- **Secure Storage**: Secrets are stored in OS-appropriate directories with proper permissions (0700/user-only)
- **YAML Format**: Easy-to-edit and audit flat String→String mappings
- **Lazy Loading**: Performance-optimized with mtime-based cache invalidation
- **No Restart Required**: Changes persist across runs without server restart
- **Environment Integration**: Integrates with the environment variable precedence system

## Secret Storage Location

Secrets are stored in platform-specific secure directories:

- **Linux/macOS**: `$XDG_DATA_HOME/wassette/secrets/` (typically `~/.local/share/wassette/secrets/`)
- **Windows**: `%APPDATA%\wassette\secrets\`

Each component's secrets are stored in a separate file named `{component-id}.yaml` with permissions set to 0700 (user-only read/write/execute).

## CLI Commands

Wassette provides three CLI commands for managing secrets:

### List Secrets

Display all secrets for a specific component:

```bash
wassette secret list <component-id>
```

**Example:**
```bash
$ wassette secret list weather-component
API_KEY: sk-abc***xyz
ENDPOINT_URL: https://api.weather.com/v1
```

**Output Format:**
- Secret keys are shown in full
- Secret values are masked with `***` in the middle for security
- Returns empty list if no secrets exist for the component

### Set a Secret

Add or update a secret for a component:

```bash
wassette secret set <component-id> <key> <value>
```

**Example:**
```bash
# Set an API key
wassette secret set weather-component API_KEY sk-1234567890abcdef

# Set multiple secrets
wassette secret set weather-component ENDPOINT_URL https://api.weather.com/v1
wassette secret set weather-component TIMEOUT 30
```

**Behavior:**
- Creates a new secret file if one doesn't exist
- Updates the value if the key already exists
- Automatically sets file permissions to 0700
- Component ID is sanitized for safe filename creation

### Delete a Secret

Remove a specific secret from a component:

```bash
wassette secret delete <component-id> <key>
```

**Example:**
```bash
$ wassette secret delete weather-component OLD_API_KEY
Secret 'OLD_API_KEY' deleted from component 'weather-component'
```

**Behavior:**
- Removes only the specified key
- Other secrets for the component remain unchanged
- Returns an error if the key doesn't exist
- Deletes the secret file if it becomes empty

## Secret File Format

Secrets are stored as YAML files with a simple flat structure:

```yaml
# ~/.local/share/wassette/secrets/weather-component.yaml
API_KEY: sk-1234567890abcdef
ENDPOINT_URL: https://api.weather.com/v1
TIMEOUT: "30"
MAX_RETRIES: "3"
```

**Important Notes:**
- All values are stored as strings
- Keys are case-sensitive
- The file is human-readable and can be manually edited
- Manual edits are automatically detected via mtime-based cache invalidation

## Environment Variable Precedence

Wassette integrates secrets into the environment variable system with the following precedence order (highest to lowest):

1. **Policy-defined environment variables** - Explicitly defined in `policy.yaml`
2. **Component secrets** - Stored via secret management system
3. **Inherited environment variables** - From the host system

This means:
- Policy-defined variables override secrets
- Secrets override inherited environment variables
- You can use secrets as defaults that can be overridden by policy

### Example Precedence

```yaml
# policy.yaml
permissions:
  environment:
    allow:
      - key: API_KEY
        value: "policy-override-key"
```

```bash
# Component secrets
$ wassette secret set my-component API_KEY "secret-key"

# Host environment
$ export API_KEY="host-key"
```

**Result**: Component receives `API_KEY="policy-override-key"` (policy wins)

## Using Secrets in Components

Components access secrets through WASI environment variable interfaces, just like regular environment variables.

### JavaScript Example

```javascript
import { get } from "wasi:config/store@0.2.0-draft";

export async function fetchWeather(city) {
    try {
        // Access secret as environment variable
        const apiKey = await get("API_KEY");
        if (!apiKey) {
            return { tag: "err", val: "API_KEY not configured" };
        }
        
        const endpoint = await get("ENDPOINT_URL") || "https://api.weather.com/v1";
        
        const response = await fetch(
            `${endpoint}/weather?q=${city}&appid=${apiKey}`
        );
        
        const data = await response.json();
        return { tag: "ok", val: JSON.stringify(data) };
    } catch (error) {
        return { tag: "err", val: error.message };
    }
}
```

### Python Example

```python
import os
import json

def fetch_data():
    """Fetch data using API key from secrets"""
    api_key = os.environ.get('API_KEY')
    if not api_key:
        return {"error": "API_KEY not configured"}
    
    endpoint = os.environ.get('ENDPOINT_URL', 'https://api.example.com')
    
    # Use api_key and endpoint for API calls
    # ...
    
    return {"status": "success"}
```

### Rust Example

```rust
use std::env;

pub fn fetch_data() -> Result<String, String> {
    let api_key = env::var("API_KEY")
        .map_err(|_| "API_KEY not configured".to_string())?;
    
    let endpoint = env::var("ENDPOINT_URL")
        .unwrap_or_else(|_| "https://api.example.com".to_string());
    
    // Use api_key and endpoint for API calls
    // ...
    
    Ok("success".to_string())
}
```

## Security Best Practices

### 1. Principle of Least Privilege

Only store secrets that components actually need:

```bash
# Good: Only necessary secrets
wassette secret set weather-component WEATHER_API_KEY sk-123

# Avoid: Unnecessary secrets
wassette secret set weather-component AWS_SECRET_KEY xxx  # Not needed by weather component
```

### 2. Rotate Secrets Regularly

Update secrets periodically to maintain security:

```bash
# Update to new API key
wassette secret set my-component API_KEY new-key-value

# Verify update
wassette secret list my-component
```

### 3. Use Policy Overrides for Sensitive Environments

For production environments, use policy files to override secrets:

```yaml
# production-policy.yaml
version: "1.0"
permissions:
  environment:
    allow:
      - key: API_KEY
        value: "production-api-key-from-vault"
```

This prevents accidentally using development secrets in production.

### 4. Audit Secret Access

Secrets are stored in user-only directories. Regularly audit access:

```bash
# Check permissions
ls -la ~/.local/share/wassette/secrets/

# Should show drwx------ (0700)
```

### 5. Don't Commit Secrets

Never commit secret files to version control:

```gitignore
# .gitignore
**/wassette/secrets/
*.secrets.yaml
```

### 6. Use Separate Secrets per Environment

Maintain different secrets for development, staging, and production:

```bash
# Development
wassette secret set app-dev API_KEY dev-key

# Production
wassette secret set app-prod API_KEY prod-key
```

## Common Workflows

### Setting Up a New Component

```bash
# 1. Load the component
wassette component load oci://ghcr.io/example/api-client:latest

# 2. Set required secrets
wassette secret set api-client API_KEY your-api-key-here
wassette secret set api-client API_ENDPOINT https://api.example.com

# 3. Grant necessary permissions
wassette permission grant network api-client api.example.com

# 4. Verify configuration
wassette secret list api-client
wassette policy get api-client
```

### Migrating from Environment Variables

If you're currently using host environment variables, migrate to secrets:

```bash
# Old approach (host env var)
export API_KEY=my-secret-key
wassette serve --stdio

# New approach (component secret)
wassette secret set my-component API_KEY my-secret-key
wassette serve --stdio
```

### Troubleshooting Secret Access

If a component reports missing secrets:

```bash
# 1. Verify secret exists
wassette secret list component-id

# 2. Check component has environment variable permission
wassette policy get component-id

# 3. Grant permission if needed
wassette permission grant environment-variable component-id API_KEY

# 4. Test access
# Component should now be able to access the secret
```

## Advanced Usage

### Programmatic Secret Management

You can manage secrets from within scripts:

```bash
#!/bin/bash
# setup-secrets.sh

COMPONENT_ID="my-component"
SECRETS_FILE="secrets.env"

# Read secrets from a file and set them
while IFS='=' read -r key value; do
    wassette secret set "$COMPONENT_ID" "$key" "$value"
done < "$SECRETS_FILE"

echo "Secrets configured for $COMPONENT_ID"
```

### Backup and Restore

Backup secrets before major changes:

```bash
# Backup
cp -r ~/.local/share/wassette/secrets/ ~/wassette-secrets-backup/

# Restore
cp -r ~/wassette-secrets-backup/ ~/.local/share/wassette/secrets/
chmod 700 ~/.local/share/wassette/secrets/
```

### Multi-Component Secret Sharing

If multiple components need the same secret, set it for each:

```bash
# Set for all components that need it
for component in component-a component-b component-c; do
    wassette secret set "$component" SHARED_API_KEY same-key-value
done
```

## Performance Considerations

- **Lazy Loading**: Secrets are loaded only when needed
- **Cache Invalidation**: Changes are detected via mtime, no polling overhead  
- **Minimal Memory**: Only active component secrets are kept in memory
- **No Server Restart**: Changes take effect immediately

## Limitations

- **Flat Structure**: Secrets are simple key-value pairs, no nested structures
- **String Values Only**: All values are stored as strings
- **No Encryption at Rest**: Secrets rely on OS file permissions for security
- **No Distributed Secrets**: Secrets are local to the machine running Wassette

For enterprise use cases requiring advanced secret management (encryption at rest, distributed secrets, etc.), consider integrating with external secret management systems like:
- HashiCorp Vault
- AWS Secrets Manager  
- Azure Key Vault
- Kubernetes Secrets

You can populate Wassette secrets from these systems using scripts or automation.

## See Also

- [Permission System](./design/permission-system.md) - Understanding Wassette's security model
- [CLI Reference](./cli.md) - Complete CLI command documentation
- [Environment Variables](./design/architecture.md#environment-variables) - Environment variable handling
- [Development Guides](./development/javascript.md) - Building components that use secrets
