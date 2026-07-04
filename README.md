# rc — the Mix shell toolkit

The rc-files that turn the [`mix`](https://github.com/markc/mix) binary
into a usable interactive login shell: aliases, functions, a prompt, ssh
shortcuts, package-manager wrappers, and helper commands. Cloned to
`~/.rc/`. It is the Mix-side analogue of
[`markc/sh`](https://github.com/markc/sh) for bash.

This repo contains **only the startup system** — no language, no
runtime. The `mix` binary comes from [`markc/mix`](https://github.com/markc/mix).

## Quick start

```sh
git clone https://github.com/markc/rc ~/.rc
cp ~/.rc/_etc/_mixrc.example ~/.mixrc
/opt/cosmix/bin/mix          # open an interactive Mix shell — toolkit loads
sshm                         # the ssh manager, now on PATH
```

(Needs the `mix` binary at `/opt/cosmix/bin/mix` first — see
[Installation](#installation).)

## Understanding rc — the mental model

Three layers, three owners:

| Layer | Where | Owner |
|---|---|---|
| The `mix` binary | `/opt/cosmix/bin/mix` | [`markc/mix`](https://github.com/markc/mix) releases |
| The shared toolkit | `~/.rc/` (this repo) | git — identical on every host |
| Machine-local config | `~/.mixrc` | you — never in git (secrets, host PATH, label) |

Everything universal lives in the repo and syncs to every host
unchanged. Everything machine-specific — secrets, extra PATH dirs,
prompt colour, personal aliases — lives in `~/.mixrc`, which the repo
never touches.

### Three-directory taxonomy

| Dir | Sourced? | PATH'd? | Purpose |
|---|---|---|---|
| `_lib/` | yes (glob) | no | Topic modules, loaded in lexical `NN-` order |
| `_bin/` | no | yes | Executable Mix scripts (`sshm`, `mx`, `f`, …) |
| `_etc/` | no, opt-in only | no | Templates to copy or source explicitly |

Nothing under `_etc/` is auto-loaded. To extend the toolkit, drop a new
`NN-name.mix` into `_lib/` — pick the `NN-` bucket by what state your
module needs from earlier ones.

### Why Mix only

Every file here is Mix source (`.mix` modules, `#!/opt/cosmix/bin/mix`
scripts). No bash fallbacks: if a Mix builtin is missing, the fix goes
into the interpreter (`cosmix-lib-mix` in the mix repo), not a shell-out
workaround here.

## Installation

Requirements: `git`, `ssh`, `rsync` (for `sshm sync`), and the `mix`
binary.

### 1. Install the mix binary

From the latest GitHub release (x86_64-linux-gnu, glibc 2.39+):

```sh
sudo install -d /opt/cosmix/bin
curl -fsSL https://github.com/markc/mix/releases/latest/download/mix \
  | sudo tee /opt/cosmix/bin/mix > /dev/null && sudo chmod 0755 /opt/cosmix/bin/mix
```

Or build from source — clone [`markc/amp`](https://github.com/markc/amp)
(the AMP protocol library family mix depends on) and
[`markc/mix`](https://github.com/markc/mix) as siblings under `$HOME`:

```sh
git clone https://github.com/markc/amp ~/.amp
git clone https://github.com/markc/mix ~/.mix
cd ~/.mix/src && cargo build --release
sudo install -m 0755 target/release/mix /opt/cosmix/bin/mix
```

`/opt/cosmix/bin/` is the canonical install location — the toolkit's
PATH setup and every script shebang assume it.

### 2. Clone the toolkit and seed your trampoline

```sh
git clone https://github.com/markc/rc ~/.rc
cp ~/.rc/_etc/_mixrc.example ~/.mixrc
```

Edit `~/.mixrc` — set `$LABEL` / `$COLOR`, add machine-local bits. Keep
it out of git.

### 3. Make mix your login shell (optional, when ready)

```sh
echo /opt/cosmix/bin/mix | sudo tee -a /etc/shells
chsh -s /opt/cosmix/bin/mix
```

### A note on PATH

Login environments differ: on PAM systems `/etc/environment` usually
injects a PATH that may or may not include `/opt/cosmix/bin`; hosts with
`UsePAM no` in sshd ignore `/etc/environment` entirely. The toolkit
doesn't care — `_lib/10-path.mix` guarantees `/opt/cosmix/bin` on PATH
whenever the directory exists, so `mix`, `which mix`, and the daemon
binaries resolve on every toolkit host regardless of how the login
environment was built.

### Running `mx` on a bash-login node

`mx <host> <cmd>` runs `ssh -q -t host "mix -i -c '<cmd>'"`, so the remote's
**login shell** has to find bare `mix` to launch it. On a mix-login-shell host
that is automatic. On a **bash-login host** — a stock Debian PVE/PBS box you
want to drive with `mx` *without* converting it to a Mix login shell —
`/opt/cosmix/bin` is not on bash's non-interactive PATH, so `mx` fails with
`mix: command not found`. The `10-path.mix` fix above only applies *inside* a
mix session, which is the chicken-and-egg: you can't start mix to fix the PATH
that lets you start mix.

`sshm sync <host>` bridges this automatically. After installing the binary it
symlinks `/opt/cosmix/bin/mix` into `/usr/local/bin` (which is on the default
PATH) whenever the remote login shell isn't already mix. That symlink is the
**entire footprint** on a bash host, and it is inert until someone types `mix`.
It does **not** touch bash's rc chain: the toolkit (`~/.rc`, `~/.mixrc`) is read
only by `mix -i`, never by bash, so a bash login stays exactly as Debian shipped
it. To undo: `rm /usr/local/bin/mix` (and optionally `rm -rf ~/.rc ~/.mixrc`).

## How the Mix shell loads rc — the source flow

```
interactive mix (login shell, `mix`, or `mix -i`)
  └─ auto-loads ~/.mixrc                 user trampoline (machine-local, NOT in git)
       └─ source ~/.rc/_mixrc            public wrapper (this repo's entry point)
            ├─ $HOME, $ostyp, $SUDO      foundational env every module relies on
            └─ glob ~/.rc/_lib/*.mix     topic modules, lexical (NN-) order
```

`_mixrc` does three things before the glob:

1. **`$HOME`** from the environment.
2. **`$ostyp`** — parses `/etc/os-release` for the distro `ID`
   (`cachyos`, `arch`, `alpine`, `debian`, `ubuntu`, …), falling back to
   `"unknown"`. Modules key OS-conditional behaviour off it.
3. **`$SUDO`** — `"/usr/bin/sudo "` when non-root, `""` when root, so
   aliases written as `${SUDO}systemctl …` work on root-only hosts with
   no sudo installed.

**Interactive only.** `~/.mixrc` loads for interactive sessions — a
login shell or `mix -i`. It does **not** load for `mix -c '…'`,
`mix script.mix`, or a plain `ssh host command`, so aliases and
functions are absent there. To run a remote command *with* the remote's
rc loaded, use [`mx`](#mx--remote-commands-with-the-remotes-rc-loaded).

### Module load order

| Module | Provides | Depends on |
|---|---|---|
| `10-path.mix` | PATH export, `path_clean()`, `path_drop()` | `$HOME` |
| `20-aliases.mix` | universal command aliases | — |
| `30-pkgmgr.mix` | per-distro package aliases | `$ostyp`, `$SUDO` |
| `40-ssh-hosts.mix` | ssh shortcut aliases | `~/.ssh/config` hosts |
| `50-tools.mix` | agent/tooling shortcuts | — |
| `60-prompt.mix` | `prompt()`, `$LABEL`/`$COLOR` defaults | — |
| `70-functions.mix` | `health`, `sc`, `es`, `newpw` | `$SUDO` |

Your `~/.mixrc` runs *after* the glob, so it can override anything the
modules set.

## Environment variables

| Variable | Set by | Meaning |
|---|---|---|
| `$HOME` | `_mixrc` | home directory |
| `$ostyp` | `_mixrc` | distro `ID` from `/etc/os-release` (`unknown` if absent) |
| `$SUDO` | `_mixrc` | `"/usr/bin/sudo "` for non-root, `""` for root |
| `PATH` | `10-path.mix` | `~/.rc/_bin : ~/.local/bin : /opt/cosmix/bin (if present) : inherited`, deduped |
| `$LABEL` | `60-prompt.mix` | prompt hostname label (default `hostname()`) |
| `$COLOR` | `60-prompt.mix` | prompt ANSI colour code (default `32`, green) |

## Everyday aliases

From `20-aliases.mix` — universal, on every toolkit host.

**Navigation and files**

| Alias | Does |
|---|---|
| `ls` / `ll` / `la` | `ls -F` / long / long+hidden, dirs first, `LC_COLLATE=C` |
| `df` | `df -kTh` |
| `disk` | `df -h` |

**Editors**

| Alias | Does |
|---|---|
| `e` | `nano -t -x -c` |
| `se` | `sudo nano -t -x -c` |

**Monitoring**

| Alias | Does |
|---|---|
| `p <pat>` | grep the full process list |
| `ports` | `ss -tuln` |
| `procs` | top 20 processes by CPU |
| `ram` | all processes by RSS (kernel threads dropped), 79-col |
| `mem` | `free -h` |
| `temp` | lm_sensors, falling back to the thermal_zone sysfs glob |
| `sysinfo` | `uname -a`, uptime, memory, root disk |
| `ff` | `fastfetch --logo none` |
| `logs` / `l` | `journalctl -f` |
| `services` | running systemd services |
| `failed` | failed systemd units |

**Git**

| Alias | Does |
|---|---|
| `gs` / `gc` / `gp` / `gd` | status / commit / push / diff |
| `gl` | `git log --oneline -20` |

## Package management — one interface, three backends

`30-pkgmgr.mix` keys off `$ostyp` so the same muscle memory works on
every distro:

| Op | Arch/CachyOS | Alpine | Debian/Ubuntu |
|---|---|---|---|
| `i <pkg>` install | `paru -S` | `apk add` | `apt-get install` |
| `r <pkg>` remove | `paru -Rns` | `apk del` | `apt-get remove --purge` |
| `s <pat>` search | `paru -Ss` | `apk search -v` | `apt-cache search` |
| `u` upgrade | `pacman -Syu` | `apk update && upgrade` | full dist-upgrade + autoremove + clean |
| `lspkg` list | `paru -Qs` | `apk info \| sort` | `dpkg -l` |
| `edpkg` edit config | `pacman.conf` | — | `sources.list` |

Arch/CachyOS additionally get upgrade tiers: `ua` (AUR only), `uu`
(repos + AUR), `uc` (upgrade + cache clean + orphan removal). All
non-self-elevating commands carry the `${SUDO}` prefix, so they work
for root and non-root alike.

## SSH host shortcuts

`40-ssh-hosts.mix` defines one-word aliases (`gw`, `mgo`, `motd`, `b1`,
`b3`, `a1`, `a2`, `a3`) that expand to `ssh <name>`. The names resolve
via `~/.ssh/config` and the `~/.ssh/hosts/*` includes that
[`sshm`](#sshm--the-ssh-manager) manages — DNS-independent. Edit this
module to match your own fleet.

## Prompt

`60-prompt.mix` defines `prompt()` — a bold, coloured
`LABEL ~/dir` prompt. `$LABEL` defaults to the hostname and `$COLOR` to
green (`32`); override both from `~/.mixrc`:

```
export LABEL = "myhost"
export COLOR = "31"
```

The current directory collapses `$HOME` to `~` (bash `\w` parity).

## Functions reference

From `70-functions.mix` and `10-path.mix`:

- **`health()`** — one-shot system snapshot: uptime, memory, root disk,
  top CPU processes.
- **`sc($action, $service)`** — `${SUDO}systemctl <action> <service>`.
- **`es`** (alias for `edit_shell()`) — edit `~/.mixrc` in nano, then
  **syntax-check it in a child mix (`mix --check`) before sourcing**, so
  a typo warns and leaves the live session intact instead of throwing a
  parse error on reload. On a clean check the file is re-sourced
  in-place — aliases, functions and exports re-register in the running
  shell, no logout needed.
- **`newpw($len)`** — random password (default 16 chars), via the Mix
  builtin rather than a `/dev/urandom` shell-out.
- **`path_clean($p)`** — dedup a `:`-separated PATH string, keeping the
  first occurrence of each segment and dropping empties.
- **`path_drop($p, $deny)`** — remove every segment listed in `$deny`
  (also `:`-separated) from a PATH string.

`path_clean`/`path_drop` exist because the toolkit glob runs *before*
your `~/.mixrc` PATH block, so both layers can prepend overlapping dirs.
The convention: `~/.mixrc` calls `path_clean()` as its **final step**,
collapsing any duplicates:

```
export PATH = path_clean(env("HOME") .. "/.cargo/bin:" .. env("PATH"))
```

## Helper commands (`_bin/`)

All are Mix scripts (`#!/opt/cosmix/bin/mix`) on PATH via `10-path.mix`.

### `f` — find files by name

```
f <pattern...>
```

Case-insensitive substring match under the current directory, long
listing. Wildcards are added for you (`f mixrc` finds `_mixrc.example`);
multiple args join with a space.

### `mx` — remote commands with the remote's rc loaded

```
mx <host> <command...>
```

The Mix twin of markc/sh's `sx` (`bash -ci`). A plain `ssh host cmd`
runs without rc — no aliases, no toolkit PATH. `mx` drives the remote
through `mix -i -c`, so the remote's `~/.mixrc` loads first:

```
mx mko ll /etc          # remote alias `ll` resolves
mx b1 'u'               # run the distro-correct upgrade alias remotely
```

Output streams live, and because `mx` allocates a remote pty
(`ssh -t`), interactive prompts — apt confirmations, sudo passwords —
work; type the answer and the remote sees it.

### `ramsum` — RSS total for matching processes

```
ramsum <pattern>
```

Lists matching processes by RSS and prints the summed total in GB.
Sibling of the `ram` alias.

### `chperms` — remote vhost permission fixer

Fixes web-vhost ownership/permissions on a remote node over ssh
(cosmix/NetServa-specific; safe to ignore unless you run that stack).
`chperms -n <node>` is a dry run.

## sshm — the SSH manager

A single tool for `~/.ssh` housekeeping: host definitions as one file
per host, Ed25519 key lifecycle, connectivity testing, toolkit
deployment, and repo updates. `sshm` alone prints short help; `sshm ha`
the full help.

| Command | Short | Does |
|---|---|---|
| `sshm init` | `i` | create the `~/.ssh` layout + `config` (idempotent) |
| `sshm create NAME IP [PORT] [USER] [KEY]` | `c` | write `~/.ssh/hosts/NAME` (defaults: 22, root, `keys/default`) |
| `sshm list` | `l` | aligned table of all hosts |
| `sshm read NAME` | `r` | show one host stanza |
| `sshm update NAME` | `u` | edit a host in `$EDITOR` (default nano) |
| `sshm delete NAME` | `d` | remove a host |
| `sshm test [NAME]` | `t` | BatchMode connectivity test — one host or all, OK/FAILED summary |
| `sshm key_create [NAME] [COMMENT] [PW]` | `kc` | Ed25519 keypair in `~/.ssh/keys/` |
| `sshm key_list` | `kl` | keys with fingerprints |
| `sshm key_read [NAME]` | `kr` | print a public key |
| `sshm key_delete NAME` | `kd` | remove a keypair |
| `sshm sync HOST` | `s` | deploy `~/.rc/` to a remote (+ bootstrap mix) |
| `sshm pull` | | fast-forward `~/.rc` from origin |
| `sshm push [MSG]` | | commit + push `~/.rc` |
| `sshm perms` | `p` | reset `~/.ssh` permissions (dirs 700, files 600) |
| `sshm start` / `sshm stop` | | start / stop+disable sshd |

## The ~/.ssh layout sshm creates

```
~/.ssh/
├── config              # generated once; Include hosts/*, ControlMaster defaults
├── authorized_keys
├── hosts/              # one file per host — created by `sshm create`
├── keys/               # Ed25519 keypairs — created by `sshm key_create`
└── mux/                # ControlMaster sockets
```

The generated `config` sets modern ciphers, keep-alives,
`IdentitiesOnly yes`, and connection multiplexing (`ControlMaster auto`,
`ControlPersist 10m`) — repeat connections to the same host reuse one
TCP session, so fleet loops run fast.

## Deploying rc to remote hosts

```
sshm sync <host>
```

Two things happen:

1. `rsync --delete` of `~/.rc/` to the remote (excluding `.git`,
   `_journal`, `_doc`) — the remote gets exactly the scaffold, nothing
   more.
2. If the remote lacks `/opt/cosmix/bin/mix`, the local binary is
   scp'd over and made executable, with a hint for the `chsh` step.

Idempotent — run it again any time the toolkit changes. The remote user
must be able to write `/opt/cosmix/bin/` for the bootstrap (root, or a
pre-created dir).

## Keeping rc updated — sshm pull and push

`sshm pull` fast-forwards `~/.rc` from origin (refusing if local is
ahead or dirty); `sshm push [message]` commits everything and pushes,
auto-generating a message from the changed paths when none is given.
After a pull on your main machine, `sshm sync <host>` distributes the
update to each remote.

If your clone lives somewhere other than `~/.rc`, set `RC_SRC` to its
path.

## Personalising with ~/.mixrc

`~/.mixrc` is the one user-edited file — both the entry point mix
auto-loads *and* your customization layer (the bash-side bashrc/myrc
split is intentionally not inherited). Start from the template:

```
-- Load the toolkit first
source env("HOME") .. "/.rc/_mixrc"

-- Machine-local PATH additions, deduped as the final step
export PATH = path_clean(env("HOME") .. "/.cargo/bin:" .. env("PATH"))

-- Prompt
export LABEL = "myhost"
export COLOR = "31"

-- Machine-local aliases
alias ai = "claude --dangerously-skip-permissions"
```

Edit it with `es` — which syntax-checks before reloading, so a typo
can't take down the live session.

**Rule of thumb:** if it should exist on every host, it belongs in a
`_lib/` module (committed); if it's this machine only — secrets, PATH,
label, personal aliases — it belongs in `~/.mixrc` (never committed).

## Troubleshooting

**`mix: not found` inside the shell, but `which mix` answers.**
`which`/`type` are REPL builtins — they know the binary's own location;
spawning `mix` as a command needs it on `$PATH`. Since `10-path.mix`
guarantees `/opt/cosmix/bin` whenever the directory exists, reload the
rc (`es`, or a fresh login) and it resolves.

**Aliases missing over `ssh host cmd`.** Expected — `~/.mixrc` loads
for interactive sessions only. Use `mx host cmd` to get the remote rc,
or `ssh host "mix -i -c '<cmd>'"` by hand.

**No PATH from `/etc/environment` on a remote.** Hosts with
`UsePAM no` in sshd skip `/etc/environment`. The toolkit compensates
(see [A note on PATH](#a-note-on-path)); daemons and cron get their
PATH elsewhere.

**`sshm pull` refuses.** Local commits or edits exist — `sshm push`
first, or resolve by hand in `~/.rc`.

**Key permissions errors.** `sshm perms` resets the whole `~/.ssh` tree
to dirs 700 / files 600.

**Which distro did the toolkit detect?** `print($ostyp)` in the shell.
`unknown` means `/etc/os-release` was missing — package aliases won't
be defined.

## Uninstalling

```sh
chsh -s /bin/bash        # if mix was your login shell
rm -rf ~/.rc ~/.mixrc    # the toolkit + your personal trampoline
```

The mix binary (`/opt/cosmix/bin/mix`) is separate — remove it (and its
`/etc/shells` line) only if nothing else on the host uses it.

## Status

Alpha, under construction. The `mix` binary it depends on is itself
under active development, so this scaffold can break in lockstep with
binary changes.

## License

MIT. Copyright (c) 2026 Mark Constable <mc@cosmix.dev>.
