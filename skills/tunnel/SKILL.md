---
name: tunnel
description: This skill should be used when the user asks to "expose localhost", "create tunnel", "share local server", "test webhooks locally", "tunnel to localhost", "make local dev accessible", "public URL for localhost", or mentions exposing local services, testing webhooks, or sharing development servers.
---

# Buddy Tunnels

Expose local services to the internet via secure Buddy tunnels. Ideal for webhook testing, client demos, mobile testing, and temporary service sharing.

## CRITICAL: AI Agent Requirements

> **STOP: Read this section completely before creating any tunnel.**

### 1. Run tunnels in background (MANDATORY)

Tunnel commands run in foreground and WILL BLOCK your execution. You MUST use `run_in_background: true` parameter in Bash tool:

```
Bash tool call:
  command: bdy tunnel http localhost:3000
  run_in_background: true
```

### 2. Ask about HTTP authentication (MANDATORY)

Before creating ANY HTTP tunnel, you MUST use AskUserQuestionTool to ask about authentication.

DO NOT skip this step. DO NOT proceed until user has made a choice.

**Question:** "Do you want to protect this HTTP tunnel with authentication?"

**Options:**
1. "HTTP Basic Auth (username:password)" → use `-a username:password` flag
2. "Buddy Authentication" → use `--buddy` flag
3. "No authentication (public access)" → proceed without auth

### 3. Docker: Verify app binds to 0.0.0.0

If running in Docker, the containerized app MUST bind to `0.0.0.0` so the host can reach it. For regular local development (not Docker), `127.0.0.1` works fine with tunnels.

## Prerequisites

**Authentication Required:** Verify with `bdy workspace ls`. If it fails, user must run `bdy login` in a separate terminal (AI agents cannot perform interactive login).

## Quick Start

### HTTP Tunnel (most common)

```bash
bdy tunnel http localhost:3000                    # basic
bdy tunnel http localhost:3000 -a user:pass       # with HTTP basic auth
bdy tunnel http localhost:3000 --buddy            # with Buddy auth
bdy tunnel http localhost:3000 -n my-tunnel       # named tunnel
```

### TCP Tunnel (databases, SSH)

```bash
bdy tunnel tcp localhost:5432    # PostgreSQL
bdy tunnel tcp localhost:3306    # MySQL
bdy tunnel tcp localhost:22      # SSH
```

### TLS Tunnel (custom certificates)

```bash
bdy tunnel tls localhost:8443 --key key.pem --cert cert.pem
```

## Common Options

| Option | Description |
|--------|-------------|
| `-n, --name` | Named tunnel for identification |
| `-a, --auth user:pass` | HTTP basic authentication |
| `--buddy` | Buddy account authentication |
| `-r, --region eu\|us\|as` | Regional endpoint |
| `-w, --whitelist` | IP CIDR restrictions |
| `-t, --timeout` | Connection timeout (seconds) |

## Troubleshooting

### Connection Refused
- Verify app is running on specified port
- If using Docker, check app binds to `0.0.0.0` inside the container

### Authentication Failed
- Run `bdy whoami` to verify auth
- User may need to run `bdy login` in separate terminal

## References

For detailed options, configurations, and examples see:
- **[references/commands.md](references/commands.md)** - Complete command reference with all flags
- **[references/examples.md](references/examples.md)** - Use cases: webhooks, demos, databases, etc.
