# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.
The human-facing manual (install, alias/command reference, troubleshooting)
is `README.md` — keep user documentation there, agent guidance here.

## What this repo is

`~/.rc/` is the **public Mix-shell rc scaffold**: the startup system
(aliases, functions, prompt, ssh shortcuts, package-manager wrappers,
helper scripts) that turns the `mix` binary into a usable interactive
login shell. Mix-side analogue of `markc/sh` for bash. No build step, no
test suite — verification is "check the syntax, source it in a fresh Mix
shell, and watch nothing explode."

The Mix **language** lives in [`markc/mix`](https://github.com/markc/mix)
(cloned to `~/.mix/`): `cosmix-mix` (binary) + `cosmix-lib-mix`
(interpreter). It builds against [`markc/amp`](https://github.com/markc/amp)
as a sibling checkout. Everything here is Mix source; the binary is under
active development, so this scaffold can break in lockstep with it.

**Before writing or editing any Mix code, read `~/.mix/AGENTS.md`** — the
dense, live-verified Mix language reference (sigils, `..` concat, string
interpolation rules, the shell-vs-Mix classifier, structured returns).
Mix has near-zero training-data presence; do not extrapolate from
bash/python. When a doc claim conflicts with live `mix -c` behaviour, the
binary is the oracle.

**Investigate Mix from the binary itself — it is self-documenting.**
Don't guess from training data; ask the binary:

- `mix builtins` — full remit of every built-in function (`mix builtins
  --json` machine-readable · `mix builtins <name>` one function · `mix
  builtins <category>` filter · or the `--builtins` flag).
- `mix man` — manual index; `mix man TOPIC` one live-verified page per
  topic (also browsable under `~/.mix/docs/_man/`).
- `mix --help` / `mix help` — CLI usage, flags, invocation modes.
- `mix -c '<probe>'` — run a one-liner to check live behaviour when a
  claim is uncertain; what the binary does wins.

## Architecture

### Load chain (interactive sessions only)

```
interactive mix (login shell / `mix -i`)
  └─ auto-loads ~/.mixrc              user trampoline (machine-local, NOT in this repo)
       └─ source ~/.rc/_mixrc         this repo's entry point
            ├─ $HOME, $ostyp, $SUDO   foundational env every module relies on
            └─ glob ~/.rc/_lib/*.mix  lexical order → NN- prefix controls sequence
```

`~/.mixrc` is intentionally not in this repo (per-machine PATH, secrets,
host aliases; template at `_etc/_mixrc.example`). `mix -c`, script runs,
and plain `ssh host cmd` do **not** load it — only interactive sessions.

### Three-directory taxonomy

| Dir | Sourced? | PATH'd? | Purpose |
|---|---|---|---|
| `_lib/` | yes (glob) | no | Topic modules, lexical (NN-) order |
| `_bin/` | no | yes (via `10-path.mix`) | Executable Mix scripts |
| `_etc/` | no, opt-in only | no | Templates the user copies explicitly |

### Load order in `_lib/`

`10-path` → `20-aliases` → `30-pkgmgr` → `40-ssh-hosts` → `50-tools` →
`60-prompt` → `70-functions`. Earlier modules establish state later ones
depend on: `10-path` exports `$PATH` (incl. `/opt/cosmix/bin` when the
dir exists) and defines `path_clean()`/`path_drop()`; `30-pkgmgr` keys
off `$ostyp` and `$SUDO` from `_mixrc`; `60-prompt` sets `$LABEL`/`$COLOR`
defaults the user's `~/.mixrc` overrides *after* the glob. To extend,
drop a new `NN-name.mix` into `_lib/`, choosing the bucket by needed
state.

### sshm specifics

`sshm pull`/`push` operate on the **toolkit repo itself** (`$REPO`,
default `~/.rc`, `RC_SRC` env override). `sshm sync <host>` rsyncs
`~/.rc/` with `--delete` (excluding `.git`, `_journal`, `_doc`) and
bootstraps `/opt/cosmix/bin/mix` on the remote if missing.

## Verifying changes

```sh
# Syntax check (parses without executing — the gate `es` uses):
/opt/cosmix/bin/mix --check ~/.rc/_lib/20-aliases.mix

# Full reload from inside a running Mix shell:
source env("HOME") .. "/.mixrc"

# Cleanest: open a fresh interactive shell and exercise the change.
/opt/cosmix/bin/mix
```

If `/opt/cosmix/bin/mix` is missing, the scaffold cannot be tested — say
so loudly (`ERROR: /opt/cosmix/bin/mix not installed`) rather than
silently falling back to bash.

## Repo-specific Mix notes

The full language reference is `~/.mix/AGENTS.md`; these are the
behaviours this codebase leans on:

- **`sh "cmd"`** runs a command via the shell: as a bare statement the
  output streams; as an expression it returns captured stdout; either way
  it sets `$rc`. For an interactive child (editor, `ssh -t`) use
  `run_stream([argv…])` — `sh`/`run` capture stdio and break TTY programs
  (see `edit_shell()` in `70-functions.mix`, `mx` in `_bin/`).
- **Assign before any conditional read.** Reading an undefined `$X` is a
  runtime error, so `if $X == nil` first-time defaults don't work —
  assign the default unconditionally (see `60-prompt.mix`).
- **Leading `~` in a double-quoted string expands to `$HOME`** at lex
  time; escape as `"\~"` for a literal tilde (prompt dir collapsing).
- **An alias whose body is a Mix call runs in the REPL process**, not a
  subshell — that's why `es` can `source ~/.mixrc` and have the reload
  land in the live session.
- **Full paths in scripts.** `run`/`run_rc`/`sh` spawn `/bin/sh` with a
  minimal PATH; call binaries as `/opt/cosmix/bin/…`, never bareword.

## Conventions to keep

- `.mix` files use `--` line comments (no `#`, no block comments).
- Each `_lib/` module starts with a header block: filename, one-line
  purpose, dependency note if it relies on earlier-module state.
- Everything in this repo must be **universal** — machine-specific
  content belongs in the user's `~/.mixrc`, never here. In particular no
  secrets, and no host material beyond the `40-ssh-hosts.mix` alias
  names.
- **No bash dependencies in `.mix` files.** A missing primitive (string
  op, formatting, RNG, …) is a Mix bug — fix it in `cosmix-lib-mix`
  (`~/.mix/src/crates/`) rather than shelling out. Genuinely-external
  tools (`ssh`, `rsync`, `find`, `ps`) are fine to shell out to.
- `~/.mixrc` PATH convention: machine rc calls `path_clean()` as its
  final step; `10-path.mix` stays distro-neutral.

## Related repos

- [`markc/mix`](https://github.com/markc/mix) — the Mix language:
  `mix` binary + interpreter library. `AGENTS.md` there is the language
  reference; agent-facing changes to Mix behaviour update it in the same
  commit.
- [`markc/amp`](https://github.com/markc/amp) — AMP protocol library
  family mix builds against (sibling checkout `~/.amp`).
- [`markc/sh`](https://github.com/markc/sh) — the bash equivalent of
  this repo; feature ports usually have a `~/.sh` original to compare
  against.
