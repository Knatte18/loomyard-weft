# A run keeps the lym binary and Bolt instructions it started with

`run-pins-its-version` · Note · enhancement, batten, stencil, config

## Seen (operator, 2026-10-27)

A run reads its instructions live while it runs: the `lym` binary, and the stencils, specs and hub config on Bolt (`_board/_lym/{stencils,specs,config}`).
A landing deploys a new `lym` and can change those files, so a run started on one version can finish on another, such as a darn run that sits between its drift rows when `darn-drift-await-row` lands and its recipe changes under it.
LoomYard takes no special care for such a run today, and the operator wants none written per change (operator, 2026-10-27).

## Idea (operator, 2026-10-27)

A run keeps the version it started with, so nothing changes under it.
- Bolt: the run records the Bolt commit at its start and reads stencils, specs and config from that commit through git, in the one place those files are read, instead of from the working files; no copy is made.
- Binary: at the run's start the running `lym` is copied into a store under a name derived from its content, as the gate's prebuilt store does (`internal/gateslot`), and the run's driver and agents call that copy.
- Shared state stays live: the board, locks, gate slots, creel mail and the run's own records.
- One verb moves a run onto the newest version, for a run that needs a fix that landed after it started.

## Open

- Shared daemons (dobby, creel, gate) run the newest binary, so a pinned run's formats must stay readable by them, and a pinned binary must not write the board in an older format.
- Plugins and skills are loaded by Claude Code from one shared place and are left out of the first step.
