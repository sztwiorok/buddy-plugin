# Buddy Package CLI Reference

Complete reference for `bdy package` commands.

## Package Management

### Create Package

```bash
bdy package create -i <name> [flags]
```

| Flag | Description |
|------|-------------|
| `-i, --name` | Package name (required) |
| `-a, --auth <user:pass>` | HTTP Basic authentication |
| `-b, --buddy` | Buddy account authentication |

**Examples:**
```bash
bdy package create -i my-site                    # Public package
bdy package create -i my-site --buddy            # Buddy auth
bdy package create -i my-site -a admin:secret    # HTTP Basic auth
```

### Get Package Info

```bash
bdy package get <name>
```

Shows package details: name, workspace, authentication type, creation date.

### Delete Package

```bash
bdy package delete <name> [flags]
```

| Flag | Description |
|------|-------------|
| `-f, --force` | Skip confirmation |

**Warning:** Deletes all versions. This is irreversible.

### Download Package

```bash
bdy package download <name>@<version> <destination>
```

Downloads a specific version to local directory.

## Version Management

### Publish Version

```bash
bdy package publish <name>@<version> <path> [flags]
```

| Flag | Description |
|------|-------------|
| `-f, --force` | Overwrite existing version |

**Examples:**
```bash
bdy package publish my-site@1.0.0 ./dist              # Publish new version
bdy package publish my-site@1.0.0 ./dist --force      # Overwrite existing
```

### List Versions

```bash
bdy package version list <name>
```

Shows all versions with their URLs, sizes, and creation dates.

### Get Version Details

```bash
bdy package version get <name> <version>
```

Shows detailed info for a specific version:
- URL
- App URL (Buddy dashboard link)
- Size
- Created timestamp

### Delete Version

```bash
bdy package version delete <name> <version> [flags]
```

| Flag | Description |
|------|-------------|
| `-f, --force` | Skip confirmation |

## URL Format

Published packages are accessible at:
```
https://<version-dashed>-<package>-<workspace>.files-pkg-2.registry.sh
```

Version dots are replaced with dashes:
- `1.0.0` → `1-0-0`
- `2.1.3` → `2-1-3`

**Example:**
Package `my-site` version `1.0.0` in workspace `myworkspace`:
```
https://1-0-0-my-site-myworkspace.files-pkg-2.registry.sh
```

## Authentication Types

| Type | Flag | Access |
|------|------|--------|
| None | (default) | Public access |
| HTTP Basic | `-a user:pass` | Username/password prompt |
| Buddy | `--buddy` | Buddy account required |

Authentication is set on package creation and applies to all versions.
