# 2026-07-03 — post-repo-split reconciliation + sshm pull/push retarget

Follow-up to the same-day PATH fix (`2e8517c` — 10-path guarantees
`/opt/cosmix/bin`; sync's empty `/etc/environment` fleet-standardised).

## What changed (`246b1b9`)

- **README.md** rewritten as a full manual in the markc/sh style
  (mental model, install, load chain, alias/pkgmgr/sshm references,
  deploy/update, troubleshooting, uninstall).
- **CLAUDE.md** retuned to agent-only guidance; the stale language
  gotchas were replaced with a pointer to `~/.mix/AGENTS.md` plus the
  repo-specific behaviours (`sh` statement-vs-expression, `run_stream`
  for TTY children, leading-tilde escape, in-REPL alias reload).
- **sshm pull/push retargeted** from the private `~/.cosmix` hub to the
  toolkit repo itself (`$REPO`, default `~/.rc`, `RC_SRC` env override;
  the old `COSMIX_SRC` var is gone). A public scaffold's git wrapper
  pointing at a private unrelated repo was incoherent — and matches the
  markc/sh `sshm` which wraps its own repo.
- Stale pre-split claims scrubbed everywhere: mix builds against
  **`~/.amp`** (markc/amp), not a `~/.cosmix` sibling; dead
  `_doc/planned/` plan paths dropped; the phantom `~/.myrc.mix` layer
  removed (one user file: `~/.mixrc`); `_mixrc.example` now shows the
  `path_clean()` final-step PATH convention.

## Fleet

- Deployed via `sshm sync` to all six nodes (cgw gcwg mko mmc sync
  obsv); interactive load + `which mix` verified on mko/mmc.
- mmc's machine `~/.mixrc` fixed: raw `/opt/cosmix/bin` prepend replaced
  with `path_clean(env("PATH"))` — the duplicate PATH entry (noted in
  the ns journal this morning) is gone.

## Verification

`mix --check` on every touched `.mix` file + sshm; `sshm ha` / `sshm
pull` smoke-tested (caught one self-inflicted leading-tilde expansion in
the new pull messages — escaped as `"\~/.rc"`).
