# rcm(1) - RC Manager

## NAME

rcm - Manage RC shell configuration deployment

## SYNOPSIS

```bash
rcm init                 # Initialize shell config on current machine
rcm sync <ssh_host>      # Sync ~/.rc to remote server
rcm help [command]       # Show help
```

## DESCRIPTION

Manages deployment of the RC shell configuration system. Use `rcm init` to set up the current machine, or `rcm sync` to deploy `~/.rc/` to remote servers.

## COMMANDS

### init

Initialize shell configuration locally.

```bash
rcm init
```

Creates `~/.myrc` from template and adds sourcing line to `~/.bashrc`.

**Safe to run multiple times.**

### sync

Deploy `~/.rc/` to remote server.

```bash
rcm sync <ssh_host>
```

Uses rsync to sync `~/.rc/` (excluding `.git`). Note: `~/.myrc` is never synced - each machine has its own.

After sync, run on remote:
```bash
ssh <host> rcm init     # Create remote's ~/.myrc
```

## FILES

- `~/.rc/_shrc` - Foundation (synced to remotes)
- `~/.myrc` - Personal config (machine-local, never synced)
- `~/.rc/_myrc.example` - Template for ~/.myrc

See `_shrc.md` for architecture and loading order details.

## EXAMPLES

### Initial Setup

```bash
git clone https://github.com/markc/rc ~/.rc
rcm init
source ~/.bashrc
```

### Deploy to Remote

```bash
rcm sync server1
ssh server1 rcm init
```

### Customize

```bash
es    # Edit ~/.myrc and reload
```

## SEE ALSO

- `_shrc.md` - Shell configuration reference
- `sshm.md` - SSH host/key management

## AUTHOR

Mark Constable <mc@netserva.org>

Copyright (C) 1995-2025 - Licensed under MIT
