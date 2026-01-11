---
name: bdy-auth
description: This skill should be used when the user asks to "install bdy", "install Buddy CLI", "login to Buddy", "authenticate with Buddy", "set up bdy", "configure Buddy authentication", or mentions Buddy CLI setup, authentication tokens, or workspace configuration.
---

# Buddy CLI Installation and Authentication

Buddy CLI (bdy) is the command-line interface for interacting with Buddy services including Sandboxes and tunnels. This skill provides installation instructions and authentication configuration for the bdy CLI.

## When to Use This Skill

Use this skill when:
- Installing the Buddy CLI for the first time
- Setting up authentication for Buddy services
- Configuring workspace and project settings
- Troubleshooting CLI authentication issues
- Switching between different Buddy accounts or regions

## Important Notes for AI Agents

**Interactive login (`bdy login`) cannot be performed by AI agents** because it requires browser interaction. If user needs authentication:
1. **Ask user to run `bdy login` in a separate terminal** - Credentials will be stored persistently
2. **Or use token-based authentication** - Via `--token` flag or environment variables
3. **Verify authentication** with `bdy workspace ls` before proceeding

## Quick Setup Workflow

### 1. Install Buddy CLI

Install bdy using the appropriate method for your platform. See [references/installation.md](references/installation.md) for detailed platform-specific instructions.

**NPM (Recommended - All Platforms):**
```bash
npm install -g bdy
```
*Requires Node.js v20 or higher*

**macOS (Homebrew):**
```bash
brew tap buddy/bdy
brew install bdy
```

**Linux (APT):**
```bash
# See references/installation.md for complete APT setup
sudo apt-get update && sudo apt-get install -y bdy
```

**Windows (Chocolatey):**
```powershell
choco install bdy --version=1.16.12-prod --pre
```

**Verify installation:**
```bash
bdy version
```

### 2. Authenticate

**IMPORTANT:** Interactive login requires browser interaction and cannot be performed by AI agents.

**Interactive Login (Recommended for Manual Setup):**

User must run this in a separate terminal session outside of the AI agent:
```bash
bdy login
```

This opens a browser for authentication and stores credentials persistently in `~/.buddy/credentials`, making them available for all terminal sessions including AI agent sessions.

**Token-Based Authentication (For AI Agents):**

If interactive login is not possible, use token-based authentication:
```bash
bdy login --token YOUR_PERSONAL_ACCESS_TOKEN
```

Or set environment variables:
```bash
export BUDDY_TOKEN="your-token"
export BUDDY_WORKSPACE="your-workspace"
export BUDDY_PROJECT="your-project"
```

For advanced authentication options including environment variables and regional configuration, see [references/authentication.md](references/authentication.md).

### 3. Verify Authentication

```bash
bdy workspace ls
```

This command lists accessible workspaces, confirming successful authentication.

## Common Configuration Options

### Workspace and Project

Set default workspace and project for CLI operations:

```bash
bdy login -w your-workspace -p your-project
```

### Regional Configuration

Buddy operates in multiple regions. Specify region during login:

```bash
bdy login --region eu  # Europe (default)
bdy login --region us  # United States
bdy login --region as  # Asia
```

### Tunnel-Specific Authentication

For tunnel operations, use the tunnel token:

```bash
bdy login --tunnel-token YOUR_TUNNEL_TOKEN
```

## Environment Variables

Configure authentication without interactive login using environment variables:

```bash
export BUDDY_TOKEN="your-token"
export BUDDY_WORKSPACE="your-workspace"
export BUDDY_PROJECT="your-project"
export BUDDY_REGION="eu"
```

For complete environment variable reference and advanced configuration options, see [references/authentication.md](references/authentication.md).

## Troubleshooting

**Authentication Failures:**
- Verify token validity in Buddy web interface
- Check workspace and project names are correct
- Ensure region matches your Buddy account region

**Command Not Found:**
- Verify installation completed successfully
- Check PATH includes bdy binary location
- Try restarting terminal/shell

**Permission Errors:**
- Ensure token has appropriate permissions
- Verify workspace and project access rights

## Additional Resources

### Reference Files

For detailed information, consult:
- **[references/installation.md](references/installation.md)** - Platform-specific installation instructions, troubleshooting
- **[references/authentication.md](references/authentication.md)** - Comprehensive authentication methods, environment variables, token management

## Next Steps

After installing and authenticating:
- Use the **sandbox** skill for deploying applications to Buddy Sandboxes
- Use the **tunnel** skill for exposing localhost services via Buddy tunnels
