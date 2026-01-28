---
name: sandbox
description: Deploy and test applications in Buddy Sandbox cloud environments. Use when user asks about "deploy app", "create sandbox", "test in cloud", "isolated environment", "remote environment", "run app in sandbox", or mentions deploying, testing, or running applications in cloud sandboxes.
---

# Buddy Sandbox Deployment

On-demand cloud environments for deploying and testing applications with public HTTP/TCP endpoints.

## Prerequisites

Verify authentication with `bdy whoami`. If it fails, user must run `bdy login` in a separate terminal.

## Workflow

### 1. Create Sandbox

**Basic creation:**
```bash
bdy sandbox create -i my-app --resources 2x4 --wait-for-running
```

**With install commands** (for dependencies):
```bash
bdy sandbox create -i my-app --resources 2x4 \
  --install-command "apt-get update && apt-get install -y nodejs npm" \
  --wait-for-configured
```

**Options:**
- Resources: `1x2`, `2x4`, `4x8`, `8x16`, `12x24` (CPUxRAM)
- `--wait-for-configured` - wait until install commands complete
- `--wait-for-running` - wait until sandbox is running, it does not wait for install commands to complete

### 2. Copy Files to Sandbox

```bash
bdy sandbox cp ./src my-app:/app > /dev/null 2>&1
```

**Redirect output to `/dev/null`** - without it, file copy floods stdout and breaks execution.

### 3. Execute Commands

Commands run in **background by default**.

**With `--wait`** for operations that must complete before next step, logs are visible in output:
```bash
bdy sandbox exec command my-app "cd /app && npm install" --wait
```

**Without `--wait`** for long-running processes (servers):
```bash
bdy sandbox exec command my-app "cd /app && npm start"
```

**Check status and logs:**
```bash
bdy sandbox exec list my-app                        # list commands
bdy sandbox exec logs my-app <command-id>           # view logs
bdy sandbox exec logs my-app <command-id> --wait    # wait for completion and show logs
```


**Creating config files:** Write locally, then copy - avoids escaping issues with heredoc through exec:
```bash
# 1. Write file locally
cat > config.json << 'EOF'
{"host": "localhost", "port": 3000}
EOF

# 2. Copy to sandbox
bdy sandbox cp config.json my-app:/app/ > /dev/null 2>&1
```

### 4. Add Endpoints (optional)

```bash
bdy sandbox endpoint add my-app -n web -e 3000
```

**Application MUST bind to `0.0.0.0`** (not `127.0.0.1` or `localhost`) - otherwise endpoint won't work.

**With authentication:**
```bash
bdy sandbox endpoint add my-app -n web -e 3000 --auth BASIC --username admin --password secret
```

**Check endpoint URL:**
```bash
bdy sandbox endpoint list my-app
```

**Reverse proxy:** Apps run behind reverse proxy. Handle `X-Forwarded-Proto` for correct HTTPS detection:
```js
// Express.js
app.set('trust proxy', true);
```
```php
// PHP
if ($_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') $_SERVER['HTTPS'] = 'on';
```

### 5. Copy Files from Sandbox

```bash
# Copy file from sandbox
bdy sandbox cp my-app:/app/result.txt ./result.txt > /dev/null 2>&1

# Copy directory from sandbox
bdy sandbox cp my-app:/app/output ./output > /dev/null 2>&1
```

**When destination already exists**, use `--merge` or `--replace`:
```bash
# Replace existing file/directory
bdy sandbox cp my-app:/app/results ./results --replace > /dev/null 2>&1

# Merge into existing directory
bdy sandbox cp my-app:/app/results ./results/ --merge > /dev/null 2>&1
```

**Tip:** Run `bdy sandbox cp --help` for detailed examples of merge/replace behavior with files and directories.           

## CRITICAL: Read Examples Before Deploying These Tech Stacks

- [Node.js](references/examples/nodejs.md)
- [Python Flask](references/examples/python-flask.md)
- [Node.js + PostgreSQL](references/examples/nodejs-postgresql.md)
- [WordPress + MySQL](references/examples/wordpress.md)
- [phpMyAdmin](references/examples/phpmyadmin.md) - database management UI for WordPress/MySQL sandbox

## References

- [Full command reference](references/commands.md)
