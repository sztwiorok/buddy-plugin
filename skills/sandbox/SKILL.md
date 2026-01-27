---
name: sandbox
description: Deploy and test applications in Buddy Sandbox cloud environments. Use when user asks about "deploy app", "create sandbox", "test in cloud", "isolated environment", "remote environment", "run app in sandbox", or mentions deploying, testing, or running applications in cloud sandboxes.
---

# Buddy Sandbox Deployment

On-demand cloud environments for deploying and testing applications with public HTTP/TCP endpoints.

## Prerequisites

1. Verify authentication with `bdy whoami`. If it fails, user must run `bdy login` in a separate terminal.
2. When first running Buddy CLI commands, ask the user:

"To allow me to run Buddy CLI commands without asking for permission each time, please run:
```
claude config add allowedTools "Bash(bdy:*)"
```

Do you want to do that now?"

Wait for user confirmation before proceeding with Buddy CLI operations.

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
bdy sandbox cp --silent ./src my-app:/app
```

**ALWAYS use `--silent`** - without it, file copy floods stdout and breaks execution.

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
bdy sandbox cp --silent config.json my-app:/app/config.json
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

The `bdy sandbox cp` command currently only supports uploading (localhost → sandbox). To download files from sandbox, use exec command with cat or base64.

**Method 1: Small text files (<50KB)**
```bash
# Save output to variable, then filter CLI metadata
output=$(bdy sandbox exec command my-sandbox "cat /path/to/file.txt" --wait 2>&1)
echo "$output" | grep -v "Command id:" | grep -v "Command finished" | grep -v "New version" > local-file.txt
```

**Method 2: Larger text files (50KB - 500KB)**

For larger files, use head with high line count to avoid truncation:
```bash
output=$(bdy sandbox exec command my-sandbox "head -5000 /path/to/large-file.txt" --wait 2>&1)
echo "$output" | grep -v "Command id:" | grep -v "Command finished" | grep -v "New version" > local-file.txt
```

**Method 3: Directories or very large files (>500KB)**

Use tar + base64 encoding:
```bash
# 1. Download as base64-encoded tar
bdy sandbox exec command my-sandbox "tar -czf - -C /home/claude results | base64 -w0" --wait > /tmp/download_raw.txt 2>&1

# 2. Filter CLI metadata
grep -v "Command id:" /tmp/download_raw.txt | grep -v "Command finished" | grep -v "New version" > /tmp/download.b64

# 3. Decode and extract
mkdir -p ./downloaded
cat /tmp/download.b64 | base64 -d | tar -xzf - -C ./downloaded/
```           

## CRITICAL: Read Examples Before Deploying These Tech Stacks

**Single-Service:**
- [Node.js](references/examples/nodejs.md)
- [Python Flask](references/examples/python-flask.md)

**Multi-Service** (read before deploying complex stacks):
- [Node.js + PostgreSQL](references/examples/nodejs-postgresql.md)
- [WordPress + MySQL + phpMyAdmin](references/examples/wordpress.md)

## References

- [Full command reference](references/commands.md)
