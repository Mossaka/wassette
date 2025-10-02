# GitHub Agentic Workflows

## Overview

Wassette uses GitHub's Agentic Workflows to automate repository maintenance, testing, and documentation tasks. These autonomous AI-powered workflows help maintain code quality and keep documentation synchronized with code changes.

## Available Workflows

### Update Docs

**File**: `.github/workflows/update-docs.md`

The Update Docs workflow is an autonomous technical writer that monitors the repository for code changes and ensures documentation remains accurate and up-to-date.

**Triggers**:
- Push to `main` branch
- Manual workflow dispatch
- Time-limited: Automatically stops after 30 days (configurable)

**Capabilities**:
- Analyzes code changes to identify new APIs, functions, or configuration changes
- Reviews existing documentation for accuracy and completeness
- Creates draft pull requests with documentation updates
- Follows documentation best practices (Diátaxis framework, Google Developer Style Guide)
- Uses active voice and progressive disclosure principles

**Configuration**:
```yaml
permissions: read-all
network: defaults
safe-outputs:
  create-pull-request:
    draft: true
tools:
  bash: [ ":*" ]
timeout_minutes: 15
```

### Weekly Research

**File**: `.github/workflows/weekly-research.md`

An autonomous research agent that performs weekly investigations into technology trends, community feedback, and ecosystem developments relevant to Wassette.

**Triggers**:
- Scheduled weekly (configurable cron schedule)
- Manual workflow dispatch
- Time-limited: Automatically stops after 30 days (configurable)

**Capabilities**:
- Researches WebAssembly ecosystem developments
- Tracks MCP (Model Context Protocol) updates
- Monitors community discussions and issues
- Investigates security best practices
- Creates summary reports as GitHub issues

**Configuration**:
```yaml
permissions: read-all
network: defaults
tools:
  web-search: enabled
  web-fetch: enabled
  github: [list_issues, create_issue, search_repositories]
```

### Daily Test Improver

**File**: `.github/workflows/daily-test-improver.md`

An autonomous testing agent that continuously improves test coverage and quality by analyzing test failures and suggesting improvements.

**Triggers**:
- Scheduled daily
- On test failures
- Manual workflow dispatch
- Time-limited: Automatically stops after 30 days (configurable)

**Capabilities**:
- Analyzes test failures and patterns
- Suggests new test cases for uncovered scenarios
- Improves existing test quality and clarity
- Identifies flaky tests
- Creates pull requests with test improvements

**Configuration**:
```yaml
permissions:
  contents: read
  actions: read
network: defaults
safe-outputs:
  create-pull-request:
    draft: true
tools:
  bash: [ ":*" ]
  github: [list_workflow_runs, get_workflow_run, list_workflow_jobs]
```

## Security Considerations

All agentic workflows implement security best practices:

### Cross-Prompt Injection Attack (XPIA) Protection

Workflows are designed to resist XPIA attacks where malicious actors embed instructions in:
- Issue descriptions or comments
- Code comments or documentation
- File contents or commit messages
- Pull request descriptions
- Web content fetched during research

**Protection Mechanisms**:
1. **Content Sanitization**: External content is treated as data, not instructions
2. **Role Boundaries**: Workflows cannot deviate from their assigned roles
3. **Action Limits**: Workflows are restricted to specific, defined actions
4. **Security Notices**: All workflows include XPIA awareness instructions

### Permission Model

Workflows follow the principle of least privilege:

- **Read-only by default**: Most workflows operate with `read-all` permissions
- **Safe outputs**: Write operations use `safe-outputs` configuration for secure PR creation
- **Network restrictions**: Network access limited to `defaults` (trusted domains)
- **Tool restrictions**: Each workflow has explicitly allowed tools

## Configuration Files

### Shared Includes

Located in `.github/workflows/agentics/shared/`:

**`include-link.md`**: Standard footer added to all workflow outputs
```markdown
> AI-generated content may contain mistakes. [Review workflow details](link)
```

**`xpia.md`**: Security notice and XPIA protection guidelines included in all workflows

### Custom Configuration

Individual workflows can be customized by creating optional config files:
- `.github/workflows/agentics/update-docs.config`
- `.github/workflows/agentics/weekly-research.config`
- `.github/workflows/agentics/daily-test-improver.config`

## Compilation Process

Agentic workflows use a markdown + YAML frontmatter format that compiles to GitHub Actions YAML:

```bash
# Compile all workflows
gh aw compile

# Compile specific workflow
gh aw compile update-docs

# View compilation output
gh aw compile --verbose
```

**Important**: Always run `gh aw compile` after modifying workflow files to generate the corresponding `.lock.yml` files.

## Monitoring and Logs

Monitor workflow execution and analyze costs:

```bash
# Download logs for all agentic workflows
gh aw logs

# Download logs for specific workflow
gh aw logs update-docs

# Filter by date range
gh aw logs --start-date -1w --end-date -1d

# Filter by engine type
gh aw logs --engine claude
```

## Best Practices

When working with agentic workflows:

1. **Review Generated Content**: Always review AI-generated pull requests before merging
2. **Set Timeouts**: Use `stop-after` directive to prevent indefinite execution
3. **Limit Permissions**: Only grant necessary permissions for each workflow
4. **Monitor Costs**: Regularly review workflow logs to track AI model usage
5. **Test Changes**: Use `workflow_dispatch` to test workflow modifications manually
6. **Document Customizations**: Document any custom configuration in workflow files

## Troubleshooting

### Workflow Not Triggering

1. Check that the workflow has been compiled: `gh aw compile update-docs`
2. Verify the `.lock.yml` file exists
3. Check if `stop-after` deadline has passed
4. Review GitHub Actions settings for repository

### Permission Errors

1. Verify workflow permissions in YAML frontmatter
2. Check that `safe-outputs` is configured for write operations
3. Ensure GITHUB_TOKEN has required permissions

### Network Access Issues

1. Check `network` configuration in workflow frontmatter
2. Verify target domains are allowed
3. Review network permission defaults

## Contributing

To propose changes to agentic workflows:

1. Modify the `.md` workflow file (not `.lock.yml`)
2. Test with `workflow_dispatch` trigger
3. Run `gh aw compile` to generate `.lock.yml`
4. Submit pull request with both files
5. Document changes in PR description

## Additional Resources

- [GitHub Agentic Workflows Documentation](https://githubnext.github.io/gh-aw/)
- [Agentic Workflows CLI Reference](https://githubnext.github.io/gh-aw/tools/cli/)
- [MCP Specification](https://modelcontextprotocol.io/)
- [Wassette Contributing Guide](../CONTRIBUTING.md)

---

*This documentation is maintained by the Update Docs workflow. For questions or improvements, please open an issue.*
