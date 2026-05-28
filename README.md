# Mix shell rc scaffold

The rc-files (aliases, functions, prompt, ssh shortcuts, package-manager
wrappers, helper binaries) that turn the `mix` binary into a usable
interactive login shell. Cloned to `~/.rc/`. Mix-side analogue of
[`markc/sh`](https://github.com/markc/sh) for bash.

This repo contains **only the startup system** — no language, no
runtime. The `mix` binary itself comes from the
[`markc/mix`](https://github.com/markc/mix) repo (currently still inside
the [`markc/cosmix`](https://github.com/markc/cosmix) monorepo as crates
`cosmix-mix` and `cosmix-lib-mix`, pending carve-out).

## Load chain

```
~/.mixrc                                user trampoline (machine-local, NOT in git)
  └─ source ~/.rc/_mixrc                public wrapper (this repo)
       ├─ sets $ostyp, $SUDO, $HOME     foundational env
       └─ globs ~/.rc/_lib/*.mix        topic modules, lexical (NN-) order
```

`~/.mixrc` is the personal trampoline — sources `_mixrc`, then adds
per-machine PATH, secrets, host aliases. Keep it out of git.

## Layout

| Path | Sourced? | PATH'd? | Purpose |
|---|---|---|---|
| `_mixrc` | yes (by `~/.mixrc`) | — | Foundational env; glob-loads `_lib/*.mix` |
| `_lib/*.mix` | yes (auto) | — | Topic modules; NN- prefix controls load order |
| `_bin/*` | — | yes | Executables (`sshm`, …) |
| `_etc/*` | no (opt-in only) | — | Templates and examples to copy/source explicitly |

Drop new `NN-name.mix` files into `_lib/` to extend; pick the NN- bucket
by what state your module needs from earlier ones.

## Install

```bash
# 1. Install the mix binary at the canonical path /opt/cosmix/bin/mix.
#    Until the mix repo is carved out, install from cosmix:
git clone https://github.com/markc/cosmix ~/.cosmix
cd ~/.cosmix/src && cargo install --path crates/cosmix-mix --root /opt/cosmix

# 2. Clone this repo:
git clone https://github.com/markc/rc ~/.rc

# 3. Seed your personal trampoline:
cp ~/.rc/_etc/_mixrc.example ~/.mixrc
# edit ~/.mixrc — set $LABEL / $COLOR, add machine-local bits.

# 4. Make mix your login shell (optional, when ready):
echo /opt/cosmix/bin/mix | sudo tee -a /etc/shells
chsh -s /opt/cosmix/bin/mix
```

## Status

Alpha, under construction. The `mix` binary it depends on is itself
under active development, so this scaffold can break in lockstep with
binary changes. Repo-split plan:
`~/.cosmix/src/_doc/planned/2026-05-24-repo-split-mix-mixrc-cos.md`.

## License

MIT. Copyright (c) 2026 Mark Constable <mc@cosmix.dev>.
