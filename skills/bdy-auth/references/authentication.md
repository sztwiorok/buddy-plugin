# Buddy CLI Authentication Guide

Comprehensive authentication methods and configuration for the Buddy CLI (bdy).

## Table of Contents
- [Authentication Methods](#authentication-methods)
- [Interactive Login](#interactive-login)
- [Token-Based Authentication](#token-based-authentication)
- [Environment Variables](#environment-variables)
- [Configuration Files](#configuration-files)
- [Regional Configuration](#regional-configuration)
- [Workspace and Project Configuration](#workspace-and-project-configuration)
- [Tunnel Authentication](#tunnel-authentication)
- [Token Management](#token-management)
- [Troubleshooting](#troubleshooting)

## Authentication Methods

Buddy CLI supports multiple authentication methods:

1. **Interactive Login** - Browser-based OAuth flow (recommended for local development, **requires manual user action**)
2. **Personal Access Token** - Token-based authentication (recommended for CI/CD and AI agents)
3. **Environment Variables** - Configuration via environment (recommended for containers and AI agents)
4. **Configuration Files** - Persistent local configuration

### For AI Agents

AI agents **cannot perform interactive login** (`bdy login`) because it requires browser interaction. When working with AI agents:
- **Option 1:** User runs `bdy login` manually in a separate terminal (credentials persist across sessions)
- **Option 2:** Use token-based authentication with `--token` flag
- **Option 3:** Set environment variables (`BUDDY_TOKEN`, etc.)

## Interactive Login

**IMPORTANT FOR AI AGENTS:** Interactive login requires browser interaction and **cannot be performed by AI agents**. Users must run `bdy login` in a separate terminal session outside of the AI agent context.

### Basic Login

```bash
bdy login
```

This opens your default browser for authentication. After successful login, credentials are stored persistently in `~/.buddy/credentials` and are available for all terminal sessions, including AI agent sessions.

### Login with Workspace

```bash
bdy login -w your-workspace
```

Authenticates and sets the default workspace.

### Login with Workspace and Project

```bash
bdy login -w your-workspace -p your-project
```

Authenticates and sets default workspace and project for subsequent commands.

### Login with Region

```bash
bdy login --region eu   # Europe (default)
bdy login --region us   # United States
bdy login --region as   # Asia
```

Specifies the Buddy region for your account.

### Complete Login Example

```bash
bdy login -w my-company -p main-project --region eu
```

Authenticates to the Europe region, setting default workspace "my-company" and project "main-project".

## Token-Based Authentication

### Using Personal Access Token

Generate a personal access token in Buddy web interface:
1. Navigate to: Settings → Personal Access Tokens
2. Click "Create Token"
3. Set permissions and expiration
4. Copy the generated token

**Authenticate with token:**

```bash
bdy login --token YOUR_PERSONAL_ACCESS_TOKEN
```

**With workspace and project:**

```bash
bdy login --token YOUR_TOKEN -w workspace -p project
```

### Token Permissions

Tokens require appropriate permissions for operations:

- **Sandboxes:** `sandbox:read`, `sandbox:write`, `sandbox:execute`
- **Tunnels:** `tunnel:read`, `tunnel:write`
- **Workspaces:** `workspace:read`
- **Projects:** `project:read`

Set minimum required permissions when creating tokens for security.

## Environment Variables

Configure authentication without interactive login using environment variables. Useful for CI/CD pipelines and containers.

### Core Environment Variables

```bash
# Required
export BUDDY_TOKEN="your-personal-access-token"

# Optional
export BUDDY_WORKSPACE="your-workspace"
export BUDDY_PROJECT="your-project"
export BUDDY_REGION="eu"  # eu, us, or as
export BUDDY_API_ENDPOINT="https://api.buddy.works"  # Override API endpoint
```

### Using Environment Variables

Once set, bdy automatically uses environment variables:

```bash
# Set environment variables
export BUDDY_TOKEN="abc123..."
export BUDDY_WORKSPACE="my-company"
export BUDDY_PROJECT="main"

# Run commands without explicit login
bdy sandbox list
bdy tunnel http localhost:3000
```

### Environment Variable Precedence

When multiple configuration methods are present, bdy uses this precedence order:

1. **Command-line flags** (highest priority)
2. **Environment variables**
3. **Configuration files**
4. **Interactive login credentials** (lowest priority)

Example:
```bash
# Environment variable sets workspace
export BUDDY_WORKSPACE="workspace-a"

# Flag overrides environment variable
bdy sandbox list -w workspace-b  # Uses workspace-b
```

### CI/CD Environment Variables

**GitHub Actions:**

```yaml
env:
  BUDDY_TOKEN: ${{ secrets.BUDDY_TOKEN }}
  BUDDY_WORKSPACE: my-company
  BUDDY_PROJECT: main
  BUDDY_REGION: eu

steps:
  - name: List sandboxes
    run: bdy sandbox list
```

**GitLab CI:**

```yaml
variables:
  BUDDY_WORKSPACE: my-company
  BUDDY_PROJECT: main
  BUDDY_REGION: eu

script:
  - export BUDDY_TOKEN=$BUDDY_TOKEN_SECRET
  - bdy sandbox list
```

**Jenkins:**

```groovy
withEnv([
    'BUDDY_TOKEN=' + env.BUDDY_TOKEN_SECRET,
    'BUDDY_WORKSPACE=my-company',
    'BUDDY_PROJECT=main',
    'BUDDY_REGION=eu'
]) {
    sh 'bdy sandbox list'
}
```

## Configuration Files

### Credentials File

After interactive login, credentials are stored in:

```
~/.buddy/credentials
```

**Format:**
```json
{
  "token": "your-token",
  "workspace": "your-workspace",
  "project": "your-project",
  "region": "eu"
}
```

### Config File

Additional configuration in:

```
~/.buddy/config
```

**Example:**
```json
{
  "default_region": "eu",
  "api_endpoint": "https://api.buddy.works",
  "timeout": 300
}
```

### Manual Configuration

Create or edit configuration files manually:

```bash
mkdir -p ~/.buddy

cat > ~/.buddy/credentials <<EOF
{
  "token": "your-token",
  "workspace": "your-workspace",
  "project": "your-project",
  "region": "eu"
}
EOF
```

## Regional Configuration

Buddy operates in multiple regions with separate API endpoints:

### Available Regions

| Region | Code | API Endpoint | Location |
|--------|------|--------------|----------|
| Europe | `eu` | `https://api.buddy.works` | Default |
| United States | `us` | `https://api.us.buddy.works` | US-based |
| Asia | `as` | `https://api.as.buddy.works` | Asia-Pacific |

### Setting Region

**During login:**
```bash
bdy login --region us
```

**Via environment variable:**
```bash
export BUDDY_REGION="us"
```

**Via API endpoint:**
```bash
export BUDDY_API_ENDPOINT="https://api.us.buddy.works"
```

### Region Selection Guidelines

- **Data Residency:** Choose region based on data compliance requirements
- **Latency:** Select region closest to your location for better performance
- **Team Location:** Use same region as your team's Buddy account

### Verifying Current Region

```bash
bdy workspace ls
```

The API endpoint used is shown in verbose output:
```bash
bdy --verbose workspace ls
```

## Workspace and Project Configuration

### Setting Default Workspace

```bash
bdy login -w your-workspace
```

Or update configuration:
```bash
export BUDDY_WORKSPACE="your-workspace"
```

### Setting Default Project

```bash
bdy login -w your-workspace -p your-project
```

Or update configuration:
```bash
export BUDDY_WORKSPACE="your-workspace"
export BUDDY_PROJECT="your-project"
```

### Overriding Defaults

Use flags to override configured defaults:

```bash
# Use different workspace for one command
bdy sandbox list -w other-workspace

# Use different project
bdy sandbox create -i test -w workspace -p project
```

### Listing Workspaces

```bash
bdy workspace ls
```

### Listing Projects

```bash
bdy project ls -w your-workspace
```

## Tunnel Authentication

Tunnels have a separate authentication token from the main Buddy token.

### Tunnel Token

Generate a tunnel token in Buddy:
1. Navigate to: Settings → Tunnel Tokens
2. Create new token
3. Copy the token

**Authenticate for tunnels:**

```bash
bdy login --tunnel-token YOUR_TUNNEL_TOKEN
```

**Or use environment variable:**

```bash
export BUDDY_TUNNEL_TOKEN="your-tunnel-token"
```

### Using Tunnel Token

Once configured, tunnel commands automatically use the tunnel token:

```bash
bdy tunnel http localhost:3000
bdy tunnel tcp localhost:5432
```

### Token Precedence for Tunnels

Tunnels use this token precedence:

1. `--token` flag on tunnel command
2. `BUDDY_TUNNEL_TOKEN` environment variable
3. Tunnel token from `~/.buddy/credentials`
4. Main `BUDDY_TOKEN` (if has tunnel permissions)

## Token Management

### Viewing Current Token

Tokens are stored in `~/.buddy/credentials`. View current configuration:

```bash
cat ~/.buddy/credentials
```

### Rotating Tokens

1. Generate new token in Buddy web interface
2. Update authentication:

```bash
bdy login --token NEW_TOKEN
```

Or update environment variable:
```bash
export BUDDY_TOKEN="NEW_TOKEN"
```

### Multiple Accounts

Manage multiple Buddy accounts using environment variables:

```bash
# Account 1
export BUDDY_TOKEN_ACCOUNT1="token1"
export BUDDY_WORKSPACE_ACCOUNT1="workspace1"

# Account 2
export BUDDY_TOKEN_ACCOUNT2="token2"
export BUDDY_WORKSPACE_ACCOUNT2="workspace2"

# Switch between accounts
alias bdy-account1='BUDDY_TOKEN=$BUDDY_TOKEN_ACCOUNT1 BUDDY_WORKSPACE=$BUDDY_WORKSPACE_ACCOUNT1 bdy'
alias bdy-account2='BUDDY_TOKEN=$BUDDY_TOKEN_ACCOUNT2 BUDDY_WORKSPACE=$BUDDY_WORKSPACE_ACCOUNT2 bdy'

# Use specific account
bdy-account1 sandbox list
bdy-account2 tunnel http localhost:3000
```

### Token Security

**Best practices:**
- Never commit tokens to version control
- Use secrets management in CI/CD (GitHub Secrets, GitLab Variables, etc.)
- Rotate tokens regularly
- Use minimum required permissions
- Store tokens in secure credential stores

**Secure token storage:**

```bash
# macOS Keychain
security add-generic-password -a "$USER" -s "buddy-token" -w "YOUR_TOKEN"
export BUDDY_TOKEN=$(security find-generic-password -a "$USER" -s "buddy-token" -w)

# Linux (using pass)
pass insert buddy/token
export BUDDY_TOKEN=$(pass buddy/token)
```

### Revoking Tokens

Revoke tokens in Buddy web interface:
1. Navigate to: Settings → Personal Access Tokens
2. Find token to revoke
3. Click "Revoke"

After revocation, re-authenticate with a new token.

## Logout

Remove stored credentials:

```bash
bdy logout
```

This deletes `~/.buddy/credentials` and requires re-authentication.

## Troubleshooting

### Invalid Token

**Symptom:** "Invalid authentication token" or "Unauthorized" errors

**Solutions:**
1. Verify token is correct:
   ```bash
   echo $BUDDY_TOKEN
   ```
2. Check token hasn't expired (check web interface)
3. Ensure token has required permissions
4. Verify token is for correct region

### Wrong Workspace/Project

**Symptom:** "Workspace not found" or "Project not found"

**Solutions:**
1. List available workspaces:
   ```bash
   bdy workspace ls
   ```
2. List projects in workspace:
   ```bash
   bdy project ls -w your-workspace
   ```
3. Check spelling and case sensitivity
4. Verify token has access to workspace/project

### Region Mismatch

**Symptom:** Authentication works but commands fail

**Solutions:**
1. Verify region matches your account:
   ```bash
   # Try different regions
   bdy login --region eu
   bdy login --region us
   bdy login --region as
   ```
2. Check which region your account is in (Buddy web interface)
3. Update region in configuration

### Tunnel Authentication Issues

**Symptom:** Tunnels fail but other commands work

**Solutions:**
1. Verify tunnel token is set:
   ```bash
   echo $BUDDY_TUNNEL_TOKEN
   ```
2. Generate new tunnel token in web interface
3. Authenticate with tunnel token:
   ```bash
   bdy login --tunnel-token NEW_TUNNEL_TOKEN
   ```

### Permission Denied

**Symptom:** "Permission denied" for specific operations

**Solutions:**
1. Check token permissions in web interface
2. Create new token with required permissions:
   - `sandbox:read`, `sandbox:write`, `sandbox:execute`
   - `tunnel:read`, `tunnel:write`
   - `workspace:read`, `project:read`
3. Re-authenticate with new token

### Configuration File Corruption

**Symptom:** Unexpected authentication behavior

**Solutions:**
1. Remove credentials and re-login:
   ```bash
   rm ~/.buddy/credentials
   bdy login
   ```
2. Verify JSON syntax if manually edited:
   ```bash
   cat ~/.buddy/credentials | jq .
   ```

### Environment Variable Not Working

**Symptom:** Environment variables don't seem to be used

**Solutions:**
1. Verify variables are exported:
   ```bash
   env | grep BUDDY
   ```
2. Check spelling (variables are case-sensitive)
3. Ensure no configuration file is overriding
4. Try unsetting and re-setting:
   ```bash
   unset BUDDY_TOKEN
   export BUDDY_TOKEN="your-token"
   ```

## Security Recommendations

1. **Use Interactive Login** for local development
2. **Use Token Authentication** for CI/CD and automation
3. **Never log tokens** in scripts or application output
4. **Rotate tokens regularly** (every 90 days recommended)
5. **Use minimum permissions** required for tasks
6. **Store tokens securely** in secrets managers
7. **Revoke unused tokens** immediately
8. **Monitor token usage** in Buddy web interface

## Quick Reference

### Interactive Login
```bash
bdy login [-w workspace] [-p project] [--region region]
```

### Token Authentication
```bash
bdy login --token TOKEN [-w workspace] [-p project]
```

### Environment Variables
```bash
export BUDDY_TOKEN="token"
export BUDDY_WORKSPACE="workspace"
export BUDDY_PROJECT="project"
export BUDDY_REGION="eu"
```

### Tunnel Authentication
```bash
bdy login --tunnel-token TOKEN
export BUDDY_TUNNEL_TOKEN="token"
```

### Logout
```bash
bdy logout
```
