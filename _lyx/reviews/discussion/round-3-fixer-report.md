# Round 3 fixer report: discussion

All three findings were validated against the code and fixed in `_lyx/discussion/decision-record.md`, with the round recorded in `_lyx/discussion/support-log.md`.

## Fixed

- **F1 (MEDIUM, design): the watcher's binary differs from the orch pane's.**
  Confirmed: `DecideStart` returns `StartSpawnWatcher` for a live strand, and `start.go` then calls `spawnWatcher` without relaunching, so the watcher runs from the current `os.Executable()`; and reed's resume loop (`lifecycle.go`) relaunches through `launchStrandLocked`, whose prelude comes from the caller's `executablePath()`.
  D4 now has reed record, on each `Strand`, the absolute path `launchStrandLocked` composed the prelude from, at every launch and relaunch, and the watcher's index source reads the orch strand's recorded path through reed at each render.
  D4 states that `start` and `--adopt` render only on a fresh launch (confirmed in `start.go`), where their binary is the pane's.
  The bound names the never-the-watcher's-own-executable rule and a strand with no recorded path.
  Scope In gains the reed record; Constraints' `PATTERN-pane-binary-resolution` line says the record comes from the same chokepoint as `LYX_BIN`; A9, the acceptance criterion and Notes for the plan writer follow; a new Open risk carries the fresh key (strands recorded before the field) per `PATTERN-review-round`.
  The support log gains the rejected watcher-executable alternative, a Round 3 entry and an updated Q14.
- **F2 (MEDIUM, design): the missing-binary bound does not hold at `startAutoReload`.**
  Confirmed: `startAutoReload` in `watcher.go` returns the `RenderRoleFile` error, and `watchloop.go` ends the watcher after `maxConsecutiveTickErrors`.
  D4 now brings `startAutoReload` in line with `startClearing` and `reloadAfterCompaction` (idle with the reason), and Scope Out admits that one change to the reload steps.
  The acceptance criterion names all three sites and rules out a failed tick; the rejected alternative and a plan-writer note record the current behaviour.
- **F3 (LOW, consistency): D9's group-`Long` rule against the root's module list.**
  Confirmed: `cmd/lyx/main.go`'s root `Long` lists the modules, and `TestCLITree_EveryCommand`'s "root.Long names every mounted module" subtest requires it.
  D9 now exempts the root, naming that subtest as the reason.

## Disputed

None.

## Deferred

None.
