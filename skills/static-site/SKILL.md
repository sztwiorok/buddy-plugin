---
name: static-site
description: Deploy static websites to Buddy Packages. Use when user asks to "deploy static site", "host static files", "publish website", "upload build artifacts", or mentions deploying pre-built HTML/CSS/JS files, SPA builds, or static exports.
---

# Buddy Static Site Deployment

Host static websites with versioning via Buddy Packages. Each version gets a unique URL.

## Prerequisites

Verify authentication with `bdy whoami`. If it fails, user must run `bdy login` in a separate terminal.

## CRITICAL: AI Agent Requirements

> **STOP: Read before publishing any package.**

### Ask about authentication (MANDATORY for new packages)

Before creating a NEW package, you MUST use AskUserQuestion tool.

**Question:** "Do you want to protect this static site with authentication?"

**Options:**
1. "HTTP Basic Auth (username:password)" → create with `-a user:pass` flag
2. "Buddy Authentication" → create with `--buddy` flag
3. "No authentication (public)" → proceed without auth flags

**Note:** Auth is set on package creation, not per-version. For existing packages, skip this question.

## Workflow

### 1. Build Static Assets (if needed)

Most frameworks need a build step:
```bash
npm run build      # Output: ./dist or ./build
```

### 2. Determine Package Name and Version

- Package name: project name or user-specified
- Version: semantic versioning (1.0.0, 1.0.1, etc.)

### 3. Create Package (first time only)

**Without auth (public):**
```bash
bdy package create my-site
```

**With auth:**
```bash
bdy package create my-site --buddy
bdy package create my-site -a admin:secret123
```

### 4. Publish Version

```bash
bdy package publish my-site@1.0.0 ./dist
```

**Shortcut (create + publish):**
```bash
bdy package publish my-site@1.0.0 ./dist --create
```

**Overwrite existing version:**
```bash
bdy package publish my-site@1.0.0 ./dist --force
```

### 5. Show Results

```bash
bdy package version get my-site 1.0.0
```

Output includes:
- `Url` - the live site URL
- `App url` - Buddy dashboard link
- `Size` - package size
- `Created` - timestamp

**URL format:**
```
https://<version>-<package>-<workspace>.files-pkg-2.registry.sh
```

Example: version `1.0.0` of package `my-site` in workspace `myplayground`:
```
https://1-0-0-my-site-myplayground.files-pkg-2.registry.sh
```

## Versioning

Each version has its own URL. Old versions remain accessible.

```bash
# List all versions
bdy package version list my-site

# Delete old version
bdy package version delete my-site 0.9.0
```

## Examples

- [Vite (React/Vue/Svelte)](references/examples/vite.md)
- [Next.js Static Export](references/examples/nextjs-static.md)
- [Plain HTML](references/examples/plain-html.md)

## References

- [Full command reference](references/commands.md)
