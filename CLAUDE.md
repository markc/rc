# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`~/.rc/` is the **public Mix-shell rc scaffold** (aliases, functions, prompt, ssh shortcuts, package-manager wrappers) that turns the `mix` binary into a usable interactive shell. It is the Mix-side analogue of `~/.sh/` (markc/sh) for bash. There is no build step and no test suite — verification is "source it in a fresh Mix shell and watch nothing explode."

The `mix` binary itself lives at [`github.com/markc/mix`](https://github.com/markc/mix) (cloned to `~/.mix/`) as `cosmix-mix` (binary) + `cosmix-lib-mix` (interpreter library), carved out of the cosmix monorepo on 2026-05-28. It is under active development; this scaffold can break in lockstep with binary changes.

## Architecture

### Load chain (read top-down)

```
~/.mixrc                            user trampoline (machine-local, NOT in this repo)
  └─ source ~/.rc/_mixrc            public wrapper (this repo's entry point)
       ├─ sets $HOME, $ostyp, $SUDO foundational env every module relies on
       └─ glob ~/.rc/_lib/*.mix     lexical order → NN- prefix controls sequence
```

`~/.mixrc` is **intentionally not** in this repo — it carries per-machine PATH, secrets, host aliases. The split mirrors `~/.sh/_shrc → _shrc.d/*.sh` discipline.

### Three-directory taxonomy

| Dir | Sourced? | PATH'd? | Purpose |
|---|---|---|---|
| `_lib/` | yes (glob) | no | Topic modules; loaded in lexical (NN-) order |
| `_bin/` | no | yes (via `10-path.mix`) | Executables on `$PATH` |
| `_etc/` | **no, opt-in only** | no | Templates, examples, snippets the user copies/sources explicitly |

Nothing under `_etc/` is auto-loaded. When adding new files, place them in the directory whose contract matches.

### Load order in `_lib/`

`10-path` → `20-aliases` → `30-pkgmgr` → `40-ssh-hosts` → `50-tools` → `60-prompt` → `70-functions`.

Earlier modules establish state later ones depend on:
- `10-path.mix` sets `$PATH` (must run before anything tries to invoke `_bin/` executables).
- `30-pkgmgr.mix` keys off `$ostyp` set in `_mixrc` — anything OS-conditional belongs after `_mixrc` runs.
- `60-prompt.mix` defines `$LABEL` / `$COLOR` defaults; user's `~/.mixrc` can override *after* the glob completes.

To extend: drop a new `NN-name.mix` into `_lib/`. Pick the NN- bucket by what state your module needs.

## Mix-language gotchas seen in this codebase

These have already bitten — preserve the workarounds, and if you trip over a new one, prefer fixing the underlying bug in `~/.mix/src/crates/cosmix-lib-mix/` over adding more workarounds here.

- **Sigil variables must be assigned before any conditional read.** Mix treats an undefined `$X` read as a runtime error, so the defensive `if $X == nil then ...` form does *not* work for first-time defaults. Assign the default unconditionally first (see `_lib/60-prompt.mix:8-9`).
- **Leading `~` in a string literal expands to `$HOME`.** To emit a literal tilde (e.g. when collapsing a path for the prompt), escape it as `"\~"` (see `_lib/60-prompt.mix:13`).
- **String interpolation uses `${VAR}`** inside double-quoted strings; concatenation is `..` (Lua-style).
- **`sh "..."` runs a subshell command**; `print` writes to stdout; `source $f` loads another `.mix` file. `function name($a, $b) ... end` defines callables.

## Verifying changes

There is no automated test suite. The realistic loops:

```bash
# Quick syntax check (parses the file without executing all side effects you'd
# get from a full shell startup):
/opt/cosmix/bin/mix -c 'source env("HOME") .. "/.rc/_lib/20-aliases.mix"'

# Full reload from inside a running Mix shell:
source env("HOME") .. "/.mixrc"

# Cleanest test: open a fresh interactive Mix shell and exercise the change.
/opt/cosmix/bin/mix
```

If `/opt/cosmix/bin/mix` is missing, the scaffold cannot be tested — say so loudly rather than silently falling back to bash.

## Conventions to keep

- `.mix` files use `--` for comments (Lua-style), not `#`.
- Module files in `_lib/` start with a header comment block: filename, one-line purpose, dependency note if it relies on state from an earlier module.
- Aliases/functions defined here must be **universal** — anything machine-specific belongs in the user's `~/.mixrc` or a sourced personal file, not in this repo.
- Don't introduce a bash dependency in a `.mix` file. If a Mix builtin is missing, add it in `cosmix-lib-mix` rather than shelling out via `sh "..."` for primitives (string ops, formatting, RNG, etc.).

## Related repos

- [`markc/mix`](https://github.com/markc/mix) — the `mix` binary (`cosmix-mix`) and interpreter library (`cosmix-lib-mix`), carved out of cosmix on 2026-05-28.
- [`markc/cosmix`](https://github.com/markc/cosmix) — the daemon-stack crates `mix` builds against pre-crates.io-publish (`cosmix-lib-amp`, `cosmix-lib-client`, `cosmix-lib-config`, `cosmix-lib-daemon`, `cosmix-lib-props`).
- [`markc/sh`](https://github.com/markc/sh) — bash equivalent of this repo.
- Repo-split plan: `~/.cosmix/src/_doc/planned/2026-05-24-repo-split-mix-mixrc-cos.md`.
