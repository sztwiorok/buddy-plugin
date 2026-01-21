# Command Reference

## Table of Contents
- [Lifecycle Commands](#lifecycle-commands)
- [Execution Commands](#execution-commands)
- [File Transfer](#file-transfer)
- [Endpoint Management](#endpoint-management)
- [Snapshot Management](#snapshot-management)
- [Command Management](#command-management)

## Lifecycle Commands

```bash
bdy sandbox create -i <identifier>     # Create sandbox
bdy sandbox list                       # List sandboxes (alias: ls)
bdy sandbox get <identifier>           # Get details
bdy sandbox status <identifier>        # Get status
bdy sandbox start <identifier>         # Start stopped sandbox
bdy sandbox stop <identifier>          # Stop running sandbox
bdy sandbox restart <identifier>       # Restart sandbox
bdy sandbox destroy <identifier>       # Delete sandbox (alias: rm)
```

### Create Options

| Option | Description |
|--------|-------------|
| `-i, --identifier <id>` | Unique identifier |
| `-n, --name <name>` | Display name |
| `--os <image>` | OS image (default: ubuntu:24.04) |
| `--resources <spec>` | Resources: 1x2, 2x4, 4x8, 8x16, 12x24 (CPUxRAM) |
| `--install-command <cmd>` | Setup command (repeatable) |
| `--run-command <cmd>` | Startup command |
| `--snapshot <name>` | Create from snapshot |

## Execution Commands

```bash
bdy sandbox exec command <identifier> "<command>"                  # Execute (background)
bdy sandbox exec command <identifier> "<command>" --wait           # Execute and wait
bdy sandbox exec command <identifier> --runtime PYTHON "<code>"    # Different runtime
bdy sandbox exec list <identifier>                                 # List commands
bdy sandbox exec logs <identifier> <command-id>                    # View logs
bdy sandbox exec logs <identifier> <command-id> -f                 # Stream logs
bdy sandbox exec kill <identifier> <command-id>                    # Kill command
```

### Exec Options

| Option | Description |
|--------|-------------|
| `--runtime <type>` | BASH (default), JAVASCRIPT, TYPESCRIPT, PYTHON |
| `--wait` | Block until command completes (default: background) |

## File Transfer

```bash
bdy sandbox cp <source> <identifier>:<dest>         # Copy to sandbox
bdy sandbox cp ./src my-app:/app/src                # Copy directory
bdy sandbox cp --silent ./file my-app:/app/file     # Silent mode (recommended)
```

**Important:** Always use `--silent` to suppress progress output.

## Endpoint Management

```bash
bdy sandbox endpoint list <identifier>                             # List (alias: ep list)
bdy sandbox endpoint get <identifier> <name>                       # Get details
bdy sandbox endpoint add <identifier> -n <name> -e <port>          # Add
bdy sandbox endpoint update <identifier> <name> -e <new-port>      # Update
bdy sandbox endpoint delete <identifier> <name>                    # Delete
```

### Endpoint Options

| Option | Description |
|--------|-------------|
| `-n, --name <name>` | Endpoint name (required) |
| `-e, --endpoint <port>` | Port number (required) |
| `-t, --type <type>` | HTTP, TLS, or TCP (default: HTTP) |
| `--auth <type>` | NONE, BASIC, or BUDDY |
| `--username/--password` | For BASIC auth |
| `--whitelist <cidr>` | IP whitelist |

## Snapshot Management

```bash
bdy sandbox snapshot list <identifier>              # List (alias: snap list)
bdy sandbox snapshot get <identifier> <name>        # Get details
bdy sandbox snapshot create <identifier> -n <name>  # Create
bdy sandbox snapshot delete <identifier> <name>     # Delete
```

## Command Management

Commands are managed via `bdy sandbox exec` subcommands (see [Execution Commands](#execution-commands)):

```bash
bdy sandbox exec list <identifier>                  # List all commands
bdy sandbox exec logs <identifier> <cmd-id>         # View logs
bdy sandbox exec logs <identifier> <cmd-id> -f      # Stream logs
bdy sandbox exec kill <identifier> <cmd-id>         # Kill command
```

**Tip:** Use `exec list` to discover command IDs, then `exec logs` to check output.
