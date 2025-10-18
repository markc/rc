# sshm(1) - SSH Manager

## NAME

**sshm** - SSH Manager for host configurations and key management

## SYNOPSIS

```bash
sshm <command> [arguments]

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
sshm help [command]
```

## DESCRIPTION

**sshm** manages SSH host configurations and SSH keys using the NetServa 3.0 directory structure:

```
~/.ssh/
├── hosts/          # Individual host config files
├── keys/           # SSH private/public key pairs
├── mux/            # SSH multiplexing sockets
└── config          # Main SSH config (includes hosts/*)
```

Each host is stored in a separate file under `~/.ssh/hosts/` and automatically included by `~/.ssh/config`.

## HOST MANAGEMENT COMMANDS

### create

Create a new SSH host configuration.

```bash
sshm create <name> <host> [port] [user] [key]
```

**Arguments:**
- `name` - SSH host alias (used with `ssh <name>`)
- `host` - Hostname or IP address
- `port` - SSH port (default: 22)
- `user` - SSH username (default: root)
- `key` - Path to identity file (default: none)

**Example:**
```bash
sshm create server1 192.168.1.100
sshm create server2 192.168.1.101 2222 admin ~/.ssh/keys/mykey
```

**Creates file:** `~/.ssh/hosts/server1`

```
Host server1
    Hostname 192.168.1.100
    Port 22
    User root
    #IdentityFile none
```

### read / show

Display SSH host configuration values.

```bash
sshm read <name>
sshm show <name>    # Alias
```

**Example:**
```bash
sshm read server1
# Output:
# server1
# 192.168.1.100
# 22
# root
# none
```

### update

Edit SSH host configuration.

```bash
sshm update <name>
```

Opens `~/.ssh/hosts/<name>` in nano editor.

**Example:**
```bash
sshm update server1
# Modify: Port, User, IdentityFile, etc.
```

### delete

Remove SSH host configuration.

```bash
sshm delete <name>
```

**Example:**
```bash
sshm delete server1
# Output: Removed: SSH host 'server1' (251)
```

**Exit codes:**
- 251 - Successfully deleted
- 255 - Host does not exist

### list

List all configured SSH hosts.

```bash
sshm list
```

**Output format:**
```
host1    192.168.1.100:22    root
host2    192.168.1.101:2222  admin
```

Shows: name, hostname:port, username

### test

Test SSH connectivity to configured hosts.

```bash
sshm test                    # Test all hosts
sshm test <name>             # Test specific host
sshm test --delete-failed    # Test all and delete unreachable
```

**Features:**
- Tests TCP connection (no authentication)
- Reports success/failure for each host
- Optional: Delete failed host configs (non-ephemeral only)
- Skips ephemeral hosts (upcloud-*, vultr-*, etc.)

**Example:**
```bash
sshm test
# Testing SSH connectivity for all hosts...
# ✓ server1 (192.168.1.100:22)
# ✗ server2 (192.168.1.101:22) - Connection failed

sshm test --delete-failed
# ... (tests all, removes unreachable non-ephemeral hosts)
```

## KEY MANAGEMENT COMMANDS

### key_create

Create a new SSH key pair.

```bash
sshm key_create <name> [comment] [password]
```

**Arguments:**
- `name` - Key name (creates ~/.ssh/keys/<name> and ~/.ssh/keys/<name>.pub)
- `comment` - Key comment (default: "$USER@$(hostname)")
- `password` - Key passphrase (default: none)

**Example:**
```bash
sshm key_create mykey "My SSH Key" "secretpass"
sshm key_create work   # No comment, no password
```

**Creates:**
- `~/.ssh/keys/mykey` (private key, 600)
- `~/.ssh/keys/mykey.pub` (public key, 644)

**Key type:** Ed25519 (modern, secure, fast)

### key_read

Display public key.

```bash
sshm key_read <name>
```

**Example:**
```bash
sshm key_read mykey
# Output: ssh-ed25519 AAAA... user@hostname
```

### key_update

Update SSH key (alias for key_create).

```bash
sshm key_update <name> [comment] [password]
```

**Note:** Overwrites existing key with same name.

### key_delete

Delete SSH key pair.

```bash
sshm key_delete <name>
```

**Example:**
```bash
sshm key_delete mykey
# Removes: ~/.ssh/keys/mykey and ~/.ssh/keys/mykey.pub
```

### key_list

List all SSH keys.

```bash
sshm key_list
```

**Output:**
```
mykey
work
personal
```

Lists private key files in `~/.ssh/keys/`.

## UTILITY COMMANDS

### init

Initialize SSH directory structure (NetServa 3.0).

```bash
sshm init
```

**Actions:**
1. Creates `~/.ssh/` (700)
2. Creates `~/.ssh/hosts/` (700)
3. Creates `~/.ssh/keys/` (700)
4. Creates `~/.ssh/mux/` (700)
5. Creates/updates `~/.ssh/config`:
   ```
   Include hosts/*
   ControlMaster auto
   ControlPath ~/.ssh/mux/%r@%h:%p
   ControlPersist 10m
   ```

**Safe to run multiple times** (idempotent).

### perms

Reset permissions for ~/.ssh directory.

```bash
sshm perms
```

**Sets:**
- `~/.ssh/` → 700
- `~/.ssh/config` → 600
- `~/.ssh/hosts/*` → 600
- `~/.ssh/keys/*` (private) → 600
- `~/.ssh/keys/*.pub` (public) → 644
- `~/.ssh/mux/` → 700

**Use when:**
- After git clone
- After rsync
- SSH complains about permissions

## FILES

### ~/.ssh/config

Main SSH configuration file.

```bash
Include hosts/*

ControlMaster auto
ControlPath ~/.ssh/mux/%r@%h:%p
ControlPersist 10m
```

Automatically includes all files in `~/.ssh/hosts/`.

### ~/.ssh/hosts/

Directory containing individual host configurations.

Each file represents one SSH host:
```
~/.ssh/hosts/server1
~/.ssh/hosts/server2
~/.ssh/hosts/server3
```

### ~/.ssh/keys/

Directory containing SSH key pairs.

```
~/.ssh/keys/mykey       (private, 600)
~/.ssh/keys/mykey.pub   (public, 644)
```

### ~/.ssh/mux/

Directory for SSH multiplexing sockets.

Enables connection sharing:
- Faster subsequent connections
- Reduced authentication overhead
- Automatic cleanup after 10m

## ARCHITECTURE

### NetServa 3.0 SSH Structure

```
~/.ssh/
├── config              # Main config (includes hosts/*)
├── hosts/              # Per-host configs
│   ├── server1
│   ├── server2
│   └── ...
├── keys/               # SSH key pairs
│   ├── mykey
│   ├── mykey.pub
│   └── ...
└── mux/                # Connection multiplexing sockets
```

**Benefits:**
- Modular: One file per host
- Version control friendly
- Easy host management
- Automatic inclusion via Include directive

## EXAMPLES

### Complete Setup Workflow

```bash
# 1. Initialize SSH structure
sshm init

# 2. Create SSH key
sshm key_create mykey "Production Key"

# 3. Create host configurations
sshm create server1 192.168.1.100 22 root ~/.ssh/keys/mykey
sshm create server2 192.168.1.101 22 admin ~/.ssh/keys/mykey

# 4. List configured hosts
sshm list

# 5. Test connectivity
sshm test

# 6. Connect
ssh server1
```

### Bulk Host Testing

```bash
# Test all hosts, remove unreachable
sshm test --delete-failed

# Verify remaining hosts
sshm list
```

### Key Management

```bash
# Create key for work
sshm key_create work "Work Laptop" "MyPassword123"

# Show public key (for authorized_keys)
sshm key_read work

# Copy to clipboard (Linux)
sshm key_read work | xclip -selection clipboard

# Copy to remote server
ssh-copy-id -i ~/.ssh/keys/work.pub user@server
```

## EXIT STATUS

- **0** - Success
- **1** - General error
- **251** - Successfully deleted
- **254** - Host/key not found
- **255** - Error (host/key doesn't exist)

## ENVIRONMENT

- `EDITOR` - Editor for update command (default: nano)

## SEE ALSO

- `ssh(1)`, `ssh-keygen(1)`, `ssh-copy-id(1)`
- `rcm(1)` - RC Manager
- `_shrc.md` - Shell configuration documentation

## BUGS

Report issues at: https://github.com/markc/rc/issues

## AUTHOR

Mark Constable <mc@netserva.org>

## COPYRIGHT

Copyright (C) 1995-2025 Mark Constable

Licensed under AGPL-3.0
