# RC — Mix Shell Configuration Toolkit

Public, shareable Mix-shell scaffold. Cloned to `~/.mix/`. Companion to
[`markc/sh`](https://github.com/markc/sh) for bash.

> **Status: alpha, under construction.** The primary dependency is the
> **mix** binary (`/opt/cosmix/bin/mix`), which is itself under
> construction at [`cosmix`](https://github.com/markc/cosmix) (crates
> `cosmix-mix` and `cosmix-lib-mix`). Until the `mix` binary repo is
> carved out as its own public release, this scaffold may break in
> lockstep with binary changes. See the
> [repo-split plan](https://github.com/markc/cosmix/blob/main/src/_doc/planned/2026-05-24-repo-split-mix-mixrc-cos.md)
> for the four-way decomposition (`mix` / `rc` / `cos` / `cosmix`).

## What this is

Mix is a pure-Rust scripting language — a best-of mix of common
scripting languages — with native **AMP (Agent Mesh Protocol)** IPC,
intended as a daily-driver replacement for bash on Cosmix hosts. This
repo is the rc-files (aliases, functions, prompt, ssh shortcuts,
package-manager wrappers) that turn the `mix` binary into a useful
interactive shell — the Mix-side analogue of how `~/.sh/` augments
bash.

## Load chain

```
~/.mixrc                                  user entry point (machine-local, NOT in git)
   └─ source ~/.mix/_mixrc                public wrapper (this repo)
         ├─ env $ostyp, $SUDO, $HOME      shell-foundational vars
         └─ glob-load ~/.mix/_mixrc.d/*.mix  topic modules (NN-prefix ordering)
```

`~/.mixrc` is the user's personal trampoline — it sources `_mixrc`, then
adds machine-local aliases/secrets/PATH extensions. The
`_mixrc → _mixrc.d/` split mirrors `~/.sh/`'s `_shrc → _shrc.d/`
discipline.

## Layout

| Path | Purpose |
|---|---|
| `_mixrc` | Public wrapper. Sets `$ostyp`, `$SUDO`, `$HOME`; glob-loads `_mixrc.d/*.mix` |
| `_mixrc.example` | Template to copy to `~/.mixrc` and edit per machine |
| `_mixrc.d/10-path.mix` | PATH setup (`~/.mix/bin/` prepend) |
| `_mixrc.d/20-aliases.mix` | Navigation, listing, common shortcuts |
| `_mixrc.d/30-pkgmgr.mix` | Per-distro package manager wrappers (`i`/`r`/`u`/`s`) |
| `_mixrc.d/40-ssh-hosts.mix` | SSH host shortcut aliases (DNS-independent) |
| `_mixrc.d/50-tools.mix` | Agent / tooling shortcuts |
| `_mixrc.d/60-prompt.mix` | Interactive REPL prompt (overridable `$LABEL` / `$COLOR`) |
| `_mixrc.d/70-functions.mix` | Helper functions (`health`, `pwgen`, etc.) |
| `bin/sshm` | SSH Manager — host/key management, init, sync, git |

Topic modules are loaded in lexical order; the `NN-` prefix controls
load sequence. Drop a new file in `_mixrc.d/` to extend.

## Installation

```bash
# 1. Install the mix binary (currently from the cosmix monorepo):
git clone https://github.com/markc/cosmix ~/.cosmix
cd ~/.cosmix/src && cargo install --path crates/cosmix-mix --root /opt/cosmix
# (when the mix binary repo is published, the install step will simplify)

# 2. Clone this repo:
git clone https://github.com/markc/rc ~/.mix

# 3. Seed your personal trampoline:
cp ~/.mix/_mixrc.example ~/.mixrc
# Edit ~/.mixrc — uncomment opt-in modules, set $LABEL / $COLOR, etc.

# 4. Make mix your login shell (optional, when ready):
chsh -s /opt/cosmix/bin/mix
```

`~/.mixrc` is intentionally **not** part of this repo — it carries
machine-local content (PATH extensions, secrets, host-specific aliases).
Keep it out of git.

## Origins

The name **Mix** comes from being a mix of common scripting languages:

- The daily-driver ergonomics of **bash** — pipelines, aliases, `$VAR`
  expansion, command-as-first-class.
- The universal-message-port idiom of **ARexx** — the AmigaOS scripting
  layer where every application exposed a named port that any script
  could address. Mix's `send`/`address`/`emit`/`on … do` forms are the
  modern AMP-over-mesh equivalent.
- The readable, beginner-friendly form of early **BASIC** dialects —
  `if / then / end`, `for / each / in`, plain control flow without
  punctuation noise.

The pure-Rust evaluator lives in the
[`cosmix`](https://github.com/markc/cosmix) monorepo under crates
`cosmix-mix` (binary, REPL, runner) and `cosmix-lib-mix` (lexer, parser,
evaluator, builtins) until it earns its own carve-out.

## License

MIT. Copyright (c) 2026 Mark Constable <mc@cosmix.dev>.
