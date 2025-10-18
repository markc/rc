# sshm(1) - SSH Manager

## NAME

sshm - Manage SSH host configurations and keys

## SYNOPSIS

```bash
# Host Management
sshm create <name> <host> [port] [user] [key]
sshm read <name>
sshm update <name>
sshm delete <name>
sshm list
sshm test [name] [--delete-failed]

# Key Management
sshm key_create <name> [comment] [password]
sshm key_read <name>
sshm key_delete <name>
sshm key_list

# Utilities
sshm init
sshm perms
```

## DESCRIPTION

Manages SSH using NetServa 3.0 directory structure with individual host files:

```
~/.ssh/
├── config          # Main config (includes hosts/*)
├── hosts/          # Individual host configs
├── keys/           # SSH key pairs
└── mux/            # Connection multiplexing sockets
```

## HOST COMMANDS

### create

```bash
sshm create server1 192.168.1.100
sshm create server2 192.168.1.101 2222 admin ~/.ssh/keys/mykey
```

Creates `~/.ssh/hosts/<name>` with Host, Hostname, Port, User, IdentityFile.

### read / show

```bash
sshm read server1
```

Display host configuration values.

### update

```bash
sshm update server1
```

Edit `~/.ssh/hosts/<name>` in nano.

### delete

```bash
sshm delete server1
```

Remove host configuration file.

### list

```bash
sshm list
```

Show all hosts: `name hostname:port user`

### test

```bash
sshm test                    # Test all hosts
sshm test server1            # Test specific host
sshm test --delete-failed    # Test all, delete unreachable
```

Tests TCP connectivity. Optionally removes failed non-ephemeral hosts.

## KEY COMMANDS

### key_create

```bash
sshm key_create mykey
sshm key_create work "Work Key" "password123"
```

Creates Ed25519 key pair in `~/.ssh/keys/`.

### key_read

```bash
sshm key_read mykey
```

Display public key.

### key_delete

```bash
sshm key_delete mykey
```

Remove key pair.

### key_list

```bash
sshm key_list
```

List all keys in `~/.ssh/keys/`.

## UTILITY COMMANDS

### init

```bash
sshm init
```

Creates NS 3.0 directory structure:
- `~/.ssh/` (700)
- `~/.ssh/hosts/` (700)
- `~/.ssh/keys/` (700)
- `~/.ssh/mux/` (700)
- `~/.ssh/config` with multiplexing

Safe to run multiple times.

### perms

```bash
sshm perms
```

Fixes all SSH permissions (use after git clone/rsync).

## EXAMPLES

### Complete Workflow

```bash
# Initialize
sshm init

# Create key
sshm key_create prod

# Add hosts
sshm create server1 192.168.1.100 22 root ~/.ssh/keys/prod
sshm create server2 192.168.1.101 22 admin ~/.ssh/keys/prod

# Test connectivity
sshm test

# Connect
ssh server1
```

### Show Public Key

```bash
sshm key_read prod
# Copy to remote: ssh-copy-id -i ~/.ssh/keys/prod.pub user@server
```

## FILES

- `~/.ssh/config` - Main config (includes hosts/*)
- `~/.ssh/hosts/*` - Individual host configs
- `~/.ssh/keys/*` - Key pairs (private 600, public 644)
- `~/.ssh/mux/*` - Multiplexing sockets

## SEE ALSO

- `rcm.md` - RC deployment
- `_shrc.md` - Shell configuration

## AUTHOR

Mark Constable <mc@netserva.org>

Copyright (C) 1995-2025 - Licensed under AGPL-3.0
