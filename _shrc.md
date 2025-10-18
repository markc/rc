# _shrc - Shell Resource Configuration

## NAME

**_shrc** - Universal shell configuration providing cross-platform aliases and functions

## DESCRIPTION

`_shrc` is the foundation of the RC system, providing portable shell enhancements that work across Alpine, Debian, Arch-based systems, and OpenWRT. It automatically detects the operating system and adapts package management aliases, service controls, and system utilities accordingly.

## ARCHITECTURE

### Loading Order

When a user logs in or starts a new shell:

```
~/.bashrc (or ~/.bash_profile)
  └─> source ~/.rc/_shrc          # 1. Foundation (this file)
        ├─> source ~/.myrc         # 2. Personal overrides (if exists)
        └─> source ~/.ns/_nsrc     # 3. NetServa additions (if exists)
```

### Sourcing Rationale

1. **`_shrc` loads FIRST** - Provides core functions that later files can use
2. **`~/.myrc` loads SECOND** - Can override `_shrc` defaults and use its functions
3. **`~/.ns/_nsrc` loads LAST** - Most specific, workstation-only additions

### Machine Types

**Remote Server:**
```
~/.rc/_shrc    # Synced from workstation
~/.myrc        # Created locally
```
Loading: `_shrc → _myrc`

**Workstation:**
```
~/.rc/_shrc    # Foundation
~/.myrc        # Personal config
~/.ns/_nsrc    # NetServa marker
```
Loading: `_shrc → _myrc → _nsrc`

## SETUP

Add to `~/.bashrc` or `~/.bash_profile`:

```bash
[[ -f ~/.rc/_shrc ]] && . ~/.rc/_shrc
```

`_shrc` automatically loads:
- `~/.myrc` (if exists)
- `~/.ns/_nsrc` (if exists)

## FEATURES

### OS Detection

Automatically detects operating system and sets:

- `OSTYP` - OS type (alpine, debian, ubuntu, cachyos, manjaro, arch, openwrt, macos)
- `ARCH` - Architecture (x86_64, arm64, armv7)

**Supported platforms:**
- Alpine Linux (apk)
- Debian/Ubuntu (apt)
- Arch/Manjaro/CachyOS (pacman/yay)
- OpenWRT (opkg)
- macOS (basic support)

### PATH Management

Sets usr-merge compatible PATH:

```bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
```

**Rationale:**
- Modern systems use symlinks: `/bin → /usr/bin`, `/sbin → /usr/sbin`
- 4 directories instead of legacy 6
- Covers all binaries via symlinks

**Extensions:**
- `~/.myrc` can extend: `export PATH=$HOME/bin:$PATH`
- `~/.ns/_nsrc` prepends: `$NS/bin:$PATH` (if exists)

### Cross-Platform Package Management

Unified aliases that adapt to OS:

| Alias | Alpine | Debian/Ubuntu | Arch | OpenWRT |
|-------|--------|---------------|------|---------|
| `i <pkg>` | apk add | apt-get install | pacman -S | opkg install |
| `r <pkg>` | apk del | apt-get remove --purge | pacman -Rns | opkg remove |
| `s <pattern>` | apk search -v | apt-cache search | pacman -Ss | opkg list |
| `u` | apk update && apk upgrade | apt-get update && apt-get dist-upgrade | pacman -Syu | opkg update && opkg upgrade |
| `lspkg` | apk info | dpkg --get-selections | pacman -Qs | opkg list-installed |

### Service Control

`sc()` function provides unified service management:

```bash
sc <action> <service>
```

**Actions:** start, stop, restart, status, enable, disable

**Adapts to:**
- systemd (`systemctl`)
- OpenRC (`rc-service`, Alpine)
- OpenWRT init scripts (`/etc/init.d/`)

**Examples:**
```bash
sc status nginx
sc restart sshd
sc enable postgresql
```

**Special:** `sc` (no args) lists all services

## ALIASES

### Navigation

- `..` - Go up one directory
- `la` - List all files with details (including hidden), sorted
- `ll` - List files with details, sorted
- `ls` - List files (colored, directories first)

### Editing

- `e` - Edit with nano (or $EDITOR)
- `se` - Edit with sudo
- `es` - Edit ~/.myrc and reload all configs

### System Information

- `ff` - Fast system info (fastfetch)
- `df` - Disk free (human readable, with filesystem types)
- `sysinfo` - System summary (uname, uptime, memory, disk)

### Monitoring

- `health` - Comprehensive system health check
- `ports` - Show listening ports (`ss -tuln`)
- `procs` - Top 20 CPU processes
- `disk` - Disk usage (human readable)
- `mem` - Memory usage (human readable)
- `ram` - Memory usage by process (sorted)
- `temp` - CPU temperature
- `top10` - Top 10 CPU processes

### Logs

- `l` - Follow system log (journalctl or syslog)
- `logs` - Follow system log (journalctl -f)
- `syslog` - Follow syslog
- `authlog` - Follow auth log
- `dlog` - PowerDNS log
- `mlog` - Mail log
- `alog` - Access log (../log/access.log)
- `elog` - Nginx error log
- `plog` - PHP error log

### Services

- `services` - List running services
- `failed` - List failed services
- `systemd` - Alias for systemctl

### Security

- `lastlog` - Recent login history
- `failedlogins` - Failed SSH attempts (last 10)
- `connections` - SSH connections
- `shblock` - Show blocked IPs (sshguard/nftables)

### Notes

- `n` - Add timestamped note to ~/.note
- `sn` - Show notes file

### Help

- `?` - Display help from ~/.help
- `m` - Show menu from ~/.menu
- `eh` - Edit help file

## FUNCTIONS

### f - Find Files

```bash
f <pattern>
```

Find files matching pattern (case-insensitive).

**Example:**
```bash
f "*.conf"
f readme
```

### sc - Service Control

```bash
sc [action] [service]
```

Cross-platform service management (described above).

### chktime - Check File Age

```bash
chktime <file> <seconds>
```

Returns 0 if file is older than specified seconds.

**Example:**
```bash
if chktime /tmp/cache 3600; then
    echo "Cache is older than 1 hour"
fi
```

### getusers - List Users

```bash
getusers
```

List users with UID between 1000-9999.

### grepuser - Search Users

```bash
grepuser <pattern>
```

Search for user in system.

### go2 - Navigate to Web Directory

```bash
go2 <vhost>
go2 <user>@<vhost>
```

Navigate to vhost directory:
- `go2 example.com` → `/srv/example.com*/web/app`
- `go2 user@example.com` → `/srv/example.com*/msg/*user`

### sx - SSH Interactive Command

```bash
sx <host> <command>
```

Execute interactive command on remote host (uses bash -ci).

**Example:**
```bash
sx server1 'echo $OSTYP'
```

### health - System Health Check

```bash
health
```

Comprehensive system health report:
- Date and uptime
- Load average
- Memory usage
- Disk usage
- Top processes
- Network connections
- Failed SSH attempts

### pstree_service - Service Process Tree

```bash
pstree_service <service>
```

Show process tree for a service.

**Example:**
```bash
pstree_service nginx
```

### shpull / shpush - Shell Repo Management

```bash
shpull           # Git pull ~/.rc/
shpush [msg]     # Git commit and push ~/.rc/
```

**Note:** Deprecated in favor of `rcm` commands.

## VARIABLES

### Set by _shrc

- `OSTYP` - OS type (alpine, debian, cachyos, etc.)
- `ARCH` - Architecture (x86_64, arm64, armv7)
- `SUDO` - Sudo command path (empty if root, `/usr/bin/sudo ` otherwise)
- `COLOR` - Prompt color (default: 31, can be overridden by ~/.myrc)
- `LABEL` - Prompt label (default: hostname, can be overridden by ~/.myrc)
- `PATH` - usr-merge optimized path

### PS1 Prompt

```bash
PS1="\[\033[1;${COLOR}m\]${LABEL} \w\[\033[0m\] "
```

Uses final values of `COLOR` and `LABEL` (after ~/.myrc overrides).

## CUSTOMIZATION

### Override in ~/.myrc

```bash
# Change prompt color and label
export COLOR=32           # Green
export LABEL=myworkstation

# Extend PATH
export PATH=$HOME/bin:$PATH

# Add personal aliases
alias myproject='cd ~/code/myapp'

# Personal functions
mybackup() {
    tar -czf backup-$(date +%Y%m%d).tar.gz "$@"
}
```

### The 'es' Alias

Quick edit and reload:

```bash
es    # Edit ~/.myrc, then reload: _shrc → _myrc → _nsrc
```

**What it does:**
1. Opens ~/.myrc in editor
2. Sources ~/.rc/_shrc (foundation)
3. Confirms: "✅ Reloaded: _shrc → _myrc → _nsrc"

## PORTABILITY

### Remote Servers

`_shrc` is designed for remote deployment:

```bash
rcm sync server1                 # Sync ~/.rc/ to remote
ssh server1 ~/.rc/rcm init       # Create remote ~/.myrc
```

**Features:**
- Minimal dependencies (bash + basic Unix tools)
- Automatic OS adaptation
- Conditional NetServa support (only if ~/.ns exists)
- Safe on servers without NetServa

### Conditional Loading

NetServa-specific features only load if `~/.ns/_nsrc` exists:

```bash
# From _shrc (line 368):
[[ -f "${NS:-$HOME/.ns}/_nsrc" ]] && . "${NS:-$HOME/.ns}/_nsrc"
```

Servers without NetServa simply skip this.

## FILES

- `~/.rc/_shrc` - This file (foundation)
- `~/.myrc` - Personal overrides (machine-local)
- `~/.ns/_nsrc` - NetServa additions (workstation only)
- `~/.bashrc` - Sources _shrc
- `~/.help` - Help file (accessed via `?` or `eh`)
- `~/.menu` - Menu file (accessed via `m`)
- `~/.note` - Notes file (accessed via `n` or `sn`)

## EXAMPLES

### Complete Setup

```bash
git clone https://github.com/markc/rc ~/.rc
~/.rc/rcm init
source ~/.bashrc
```

### Customize

```bash
es    # Edit ~/.myrc

# Add your customizations:
export LABEL=$(hostname)
export COLOR=36  # Cyan
alias ll='ls -lah'
alias update='sudo apt update && sudo apt upgrade'
```

### Deploy to Remote

```bash
rcm sync myserver
ssh myserver ~/.rc/rcm init
ssh myserver
# _shrc automatically loaded with OSTYP detected
```

## OS-SPECIFIC NOTES

### Alpine Linux

- Uses `apk` package manager
- Uses OpenRC service manager
- `sc()` uses `rc-service` and `rc-update`

### Debian/Ubuntu

- Uses `apt` package manager
- Uses systemd service manager
- `sc()` uses `systemctl`

### Arch/Manjaro/CachyOS

- Uses `pacman` package manager
- Includes `yay` support (via `uu` alias)
- Uses systemd service manager

### OpenWRT

- Uses `opkg` package manager
- Uses init.d scripts
- `sc()` uses `/etc/init.d/` directly
- Limited function exports to reduce memory

## ENVIRONMENT INTEGRATION

### NetServa Platform

When `~/.ns/_nsrc` exists (workstation):

```bash
# Added by _nsrc:
export NS="$HOME/.ns"
export NETSERVA_VERSION="3.0.0"
export NS_SUCCESS="✅"
# ... (emoji variables)
alias a='php artisan'
alias c='composer'
# ... (NetServa functions)
```

## SEE ALSO

- `rcm(1)` - RC Manager for deployment
- `sshm(1)` - SSH Manager for host/key management
- `bash(1)` - GNU Bourne-Again SHell

## BUGS

Report issues at: https://github.com/markc/rc/issues

## AUTHOR

Mark Constable <mc@netserva.org>

## COPYRIGHT

Copyright (C) 1995-2025 Mark Constable

Licensed under MIT License (see file header)
