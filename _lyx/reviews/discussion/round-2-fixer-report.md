# Discussion round 2 — fixer report

All seven findings were validated against the code and fixed in `_lyx/discussion/decision-record.md`; `_lyx/discussion/support-log.md` gains a round-2 entry.

## Fixed

- **F1 (BLOCKING, D7).**
  Reproduced on tmux 3.6 with `pane-border-status top`: `resize-pane -y 3` on the top pane gives a 4-row cell (`80x4`), on a lower pane a 3-row cell.
  `render.FixedHeightPins` pins cell heights.
  D7 now keeps every planned cell height, pins the pane at window row 0 at its cell height minus the title row, keeps every other pin (Selvage's included) at its cell height, and names the Selvage-only window as pin-free.
  The acceptance criterion checks per-cell heights after a resize with the hook installed.
- **F2 (BLOCKING, D13).**
  Confirmed: `reedengine.LoadConfig` (`config.go:51`) only unmarshals, and `mouse`/`watchdog` are validated in `ensureServerAndSessionLocked` (`lifecycle.go:240`).
  `LoadConfig` is also called by pair teardown, burler wiring and the orch CLI.
  D13 now refuses an out-of-palette color in that pre-tmux validation block and makes every other reader fail safe to no color, logged.
  The constraint line and the acceptance criterion match.
  The criterion names "an attach to a live session", since an attach that cold-boots passes through the same validation block.
- **F3 (MEDIUM, D12).**
  The Bound now says the order holds across a full restart and against not-live predecessors, but not against a still-live later session or a failed predecessor.
  The acceptance criterion says a failed pair's session ends up out of order, and the Open risks entry points at the Bound instead of restating it.
- **F4 (MEDIUM, D12).**
  D12 states the lock sequence: read the trigger under the booter's lock, release it, run the ups, re-take it, and re-run the whole session-ensuring step.
  It also states what each list entry carries: name, spawn-order key, and an up function the teller built from that worktree's own `reed.yaml` and geometry.
- **F5 (MEDIUM, D15).**
  A file whose permission bits deny the user write access is skipped before any write.
  The temp file gets the target's permission bits.
  The acceptance criterion matches.
- **F6 (LOW, D15).**
  Only one begin then one end, inside the top-level array, is accepted, and every other marker shape is a skip.
  Inserted lines use the file's existing line ending, and the block ends with a line terminator.
- **F7 (LOW, D13/D14).**
  A segment with no config key is no color, logged.
  The resolved palette color is carried on `reedengine.StrandStatus`, which the orch CLI already reads through `Status` (`orchcli/cli.go`), and the watcher takes it from there.

## Disputed

None.

## Deferred

None.
