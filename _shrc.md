# _shrc - Shell Resource Configuration

## NAME

_shrc - Cross-platform shell configuration foundation

## DESCRIPTION

Universal shell configuration providing portable aliases, functions, and utilities. Automatically detects OS and adapts package management, service control, and system tools for Alpine, Debian, Arch, and OpenWRT.

## ARCHITECTURE

### File Structure

```
~/.rc/_shrc        # Foundation (this file, synced to remotes)
~/.myrc            # Personal config (machine-local, never synced)
~/.ns/_nsrc        # NetServa additions (workstation only, optional)
```

### Loading Order

```bash
~/.bashrc
  └─> source ~/.rc/_shrc          # 1. Foundation
        ├─> source ~/.myrc         # 2. Personal (if exists)
        └─> source ~/.ns/_nsrc     # 3. NetServa (if exists)
```

**Rationale:**
1. `_shrc` loads FIRST - provides functions/aliases for later files
2. `~/.myrc` loads SECOND - can override defaults, use _shrc functions
3. `~/.ns/_nsrc` loads LAST - workstation-specific NetServa additions

### Machine Types

**Remote Server:**
- Has: `~/.rc/_shrc`, `~/.myrc`
- Loading: `_shrc → _myrc`

**Workstation:**
- Has: `~/.rc/_shrc`, `~/.myrc`, `~/.ns/_nsrc`
- Loading: `_shrc → _myrc → _nsrc`

### Variables Set

**By _shrc:**
- `OSTYP` - OS type (alpine, debian, cachyos, openwrt)
- `ARCH` - Architecture (x86_64, arm64, armv7)
- `SUDO` - Sudo path (empty if root)
- `COLOR` - Prompt color (31, overridable in ~/.myrc)
- `LABEL` - Prompt label (hostname, overridable in ~/.myrc)
- `PATH` - usr-merge: `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin`

**By _nsrc (workstation only):**
- `NS` - NetServa directory
- `NETSERVA_VERSION` - Version
- `NS_SUCCESS`, `NS_ERROR`, etc. - Status emojis

## SETUP

Add to `~/.bashrc`:

```bash
[[ -f ~/.rc/_shrc ]] && . ~/.rc/_shrc
```

_shrc automatically sources `~/.myrc` and `~/.ns/_nsrc` if they exist.

## KEY FEATURES

### Cross-Platform Package Management

Unified aliases adapt to OS:

```bash
i <pkg>      # Install (apk add, apt install, pacman -S, opkg install)
r <pkg>      # Remove
s <pattern>  # Search
u            # Update system
lspkg        # List installed
```

### Service Control

```bash
sc <action> <service>    # Works across systemd/OpenRC/OpenWRT
sc                       # List all services
```

Actions: start, stop, restart, status, enable, disable

### Essential Aliases

**Navigation:** `..`, `la`, `ll`
**Editing:** `e`, `se`, `es` (edit ~/.myrc and reload)
**Monitoring:** `health`, `ports`, `procs`, `disk`, `mem`, `ram`
**Logs:** `l`, `logs`, `syslog`, `authlog`
**System:** `ff`, `sysinfo`, `services`, `failed`

### Essential Functions

```bash
f <pattern>              # Find files
health                   # System health check
pstree_service <svc>     # Show service process tree
go2 <vhost>              # Navigate to /srv/<vhost>/web/app
sx <host> <cmd>          # SSH interactive command
```

## CUSTOMIZATION

Edit `~/.myrc`:

```bash
es    # Edit and reload
```

Example `~/.myrc`:

```bash
# Essential aliases
alias rcm=~/.rc/rcm
alias sshm=~/.rc/sshm

# Customize prompt
export LABEL=myworkstation
export COLOR=32

# Extend PATH
export PATH=$HOME/bin:$PATH

# Personal aliases
alias myproject='cd ~/code/myapp'
```

## OS ADAPTATION

### Detected Platforms

- Alpine (apk, OpenRC)
- Debian/Ubuntu (apt, systemd)
- Arch/Manjaro/CachyOS (pacman, systemd)
- OpenWRT (opkg, init.d)

### Package Manager Aliases

| OS | Install | Remove | Search | Update |
|----|---------|--------|--------|--------|
| Alpine | apk add | apk del | apk search | apk update && upgrade |
| Debian | apt install | apt remove | apt search | apt update && dist-upgrade |
| Arch | pacman -S | pacman -Rns | pacman -Ss | pacman -Syu |
| OpenWRT | opkg install | opkg remove | opkg list | opkg update && upgrade |

### Service Control Adaptation

| OS | Implementation |
|----|---------------|
| systemd | `systemctl <action> <service>` |
| OpenRC | `rc-service <service> <action>` |
| OpenWRT | `/etc/init.d/<service> <action>` |

## DEPLOYMENT

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

## FILES

- `~/.rc/_shrc` - This file
- `~/.myrc` - Personal config (created by `rcm init`)
- `~/.ns/_nsrc` - NetServa marker (workstation only)

## SEE ALSO

- `rcm.md` - RC Manager for deployment
- `sshm.md` - SSH host/key management

## AUTHOR

Copyright (C) 1995-2025 Mark Constable <mc@netserva.org> (MIT License)
