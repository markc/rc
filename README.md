# RC - Shell Configuration Toolkit

Portable shell configuration providing cross-platform aliases, functions, and tools for system administration. Part of the NetServa 3.0 ecosystem.

## Installation

```bash
git clone https://github.com/markc/rc ~/.rc
echo '[[ -f ~/.rc/_shrc ]] && . ~/.rc/_shrc' >> ~/.bashrc
source ~/.bashrc
```

## Architecture

```
~/.bashrc          # Sources ~/.rc/_shrc
~/.rc/_shrc        # Universal shell toolkit (synced to remotes)
~/.myrc            # Personal config (machine-local, never synced)
~/.ns/_nsrc        # NetServa workstation marker (if present)
```

**Loading order:** `_shrc` → `_myrc` → `_nsrc` → PS1

## Key Features

- **Cross-platform**: Alpine, Debian, Arch-based, OpenWRT
- **OS detection**: Automatic package manager aliases (apk, apt, pacman, opkg)
- **Service control**: `sc()` function works across systemd/OpenRC/OpenWRT
- **usr-merge ready**: Optimized PATH for modern systems
- **NetServa aware**: Loads `~/.ns/_nsrc` if workstation

## Essential Aliases & Functions

**Navigation:** `..` `la` `ll` `f <pattern>`
**Editing:** `e` `se` `es`
**Package Mgmt:** `i` `r` `s` `u` `lspkg`
**Services:** `sc <action> <service>`
**Monitoring:** `health` `ports` `procs` `disk` `mem` `logs`
**System:** `ff` `sysinfo` `ram` `services` `failed`

## Personal Configuration

Create `~/.myrc` for machine-local customizations:

```bash
~/.rc/rcm init  # Creates ~/.myrc from template
```

Your `~/.myrc` can:
- Override COLOR/LABEL
- Extend PATH: `export PATH=$HOME/bin:$PATH`
- Add personal aliases and functions
- Store API tokens (never synced/versioned)

## Remote Deployment

```bash
~/.rc/rcm sync <ssh_host>    # Syncs ~/.rc to remote
ssh <host> ~/.rc/rcm init    # Creates remote's ~/.myrc
```

**Note:** `~/.myrc` is never synced - each machine has its own.

## Tools

**rcm** - RC manager (init, sync)
**sshm** - SSH host/key manager

## Supported Platforms

- Debian/Ubuntu (apt)
- Arch/Manjaro/CachyOS (pacman/yay)
- Alpine Linux (apk)
- OpenWRT (opkg)

## License

Copyright (C) 1995-2025 Mark Constable <mc@netserva.org> (MIT License)
