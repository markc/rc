# rcm(1) - RC Manager

## NAME

**rcm** - Runtime Configuration (RC) manager for shell environment

## SYNOPSIS

```bash
rcm <command> [arguments]
rcm init
rcm sync <ssh_host>
rcm help [command]
```

## DESCRIPTION

**rcm** manages the Runtime Configuration (RC) system, a portable shell configuration toolkit that provides cross-platform aliases, functions, and tools for system administration.

The RC system consists of:
- `~/.rc/_shrc` - Universal shell toolkit (synced across machines)
- `~/.myrc` - Personal configuration (machine-local, never synced)
- `~/.ns/_nsrc` - NetServa Platform marker (workstation only)

## COMMANDS

### init

Initialize shell configuration on local machine.

```bash
rcm init
```

**Actions:**
1. Creates `~/.myrc` from `~/.rc/_myrc.example` template (if missing)
2. Adds sourcing line to `~/.bashrc` (if not present)
3. Reports status

**Output:**
- ✅ Created ~/.myrc from template
- ✅ Added sourcing line to ~/.bashrc
- ℹ️  Run: source ~/.bashrc (or restart shell)

**Notes:**
- Safe to run multiple times (idempotent)
- Will not overwrite existing `~/.myrc`
- Each machine has its own independent `~/.myrc`

### sync

Synchronize `~/.rc/` directory to remote server.

```bash
rcm sync <ssh_host>
```

**Arguments:**
- `ssh_host` - SSH host configured in `~/.ssh/config` or `~/.ssh/hosts/`

**Actions:**
1. Validates SSH host exists
2. Syncs `~/.rc/` to remote using rsync
3. Excludes `.git` directory
4. Does NOT sync `~/.myrc` (machine-local only)

**Example:**
```bash
rcm sync myserver
ssh myserver ~/.rc/rcm init  # Initialize on remote
```

**Notes:**
- Uses `rsync -avz` for efficient transfer
- Preserves permissions and timestamps
- Each remote needs its own `rcm init` after sync
- `~/.myrc` remains machine-local (never synced)

### help

Display help information.

```bash
rcm help           # General help
rcm help init      # Help for init command
rcm help sync      # Help for sync command
```

## FILES

### ~/.rc/_shrc

Universal shell configuration (synced).

Contains:
- Cross-platform aliases
- Functions (f, sc, health, etc.)
- OS detection and adaptation
- Package manager aliases
- Service control wrappers

### ~/.myrc

Personal configuration (machine-local, never synced).

Use for:
- API tokens and credentials
- Machine-specific aliases
- Color/label customization
- PATH extensions: `export PATH=$HOME/bin:$PATH`

Created by: `rcm init`

### ~/.rc/_myrc.example

Template for personal configuration.

Minimal header-only file used by `rcm init` to create `~/.myrc`.

### ~/.ns/_nsrc

NetServa Platform configuration (workstation only).

Provides:
- NetServa environment variables (NS_*, NETSERVA_VERSION)
- NetServa functions (ns_db, ns_log, nspull, nspush)
- NetServa aliases (a, c, lx)
- Adds `~/.ns/bin` to PATH (if exists)

## ARCHITECTURE

### Loading Order

```
~/.bashrc
  └─> ~/.rc/_shrc (foundation)
        ├─> ~/.myrc (personal overrides)
        └─> ~/.ns/_nsrc (NetServa additions, if exists)
```

### Machine Types

**Remote Server:**
```
~/.rc/_shrc    # Synced from workstation
~/.myrc        # Created locally via "rcm init"
```

**Workstation:**
```
~/.rc/_shrc    # Foundation
~/.myrc        # Personal config
~/.ns/_nsrc    # NetServa marker
```

## ENVIRONMENT

### Variables Set by _shrc

- `OSTYP` - Detected OS (alpine, debian, cachyos, openwrt, etc.)
- `ARCH` - Architecture (x86_64, arm64, armv7)
- `SUDO` - Sudo command path (if not root)
- `COLOR` - Prompt color (default: 31)
- `LABEL` - Prompt label (default: hostname)
- `PATH` - usr-merge optimized: `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin`

### Variables Set by _nsrc (Workstation Only)

- `NS` - NetServa directory (`$HOME/.ns`)
- `NETSERVA_VERSION` - Version (e.g., "3.0.0")
- `NS_DB` - SQLite database path
- `NS_SUCCESS`, `NS_ERROR`, `NS_WARN`, etc. - Status emojis

## EXAMPLES

### Initial Setup (New Machine)

```bash
git clone https://github.com/markc/rc ~/.rc
~/.rc/rcm init
source ~/.bashrc
```

### Deploy to Remote Server

```bash
# Sync configuration
rcm sync server1

# Initialize on remote
ssh server1 ~/.rc/rcm init

# Verify
ssh server1 'echo $OSTYP'
```

### Customize Personal Config

```bash
# Edit personal configuration
nano ~/.myrc

# Add custom settings
export LABEL=myworkstation
export COLOR=32
alias myproject='cd ~/code/myapp'

# Reload (use 'es' alias or manual)
source ~/.rc/_shrc
```

## EXIT STATUS

- **0** - Success
- **1** - Error (missing arguments, command failed, etc.)

## SEE ALSO

- `sshm(1)` - SSH Manager for host/key management
- `_shrc.md` - Detailed _shrc documentation
- `README.md` - Project overview

## BUGS

Report issues at: https://github.com/markc/rc/issues

## AUTHOR

Mark Constable <mc@netserva.org>

## COPYRIGHT

Copyright (C) 1995-2025 Mark Constable

Licensed under AGPL-3.0
