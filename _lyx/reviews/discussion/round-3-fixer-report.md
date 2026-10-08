# Discussion round 3 — fixer report

Every finding was checked against the code and the record, and all seven hold.
All changes are in `_lyx/discussion/decision-record.md`; `support-log.md` is unchanged, since its round-2 ledger entries are history and do not contradict the new text.

## Fixed

- **F1 (MEDIUM, design).**
  Confirmed: `render.Pin` carries only `PaneID` and `Height`, and `FixedHeightPins` takes no `paneOrder` (`internal/reedengine/render/rules.go`), while `Rules` resequences cells by `paneOrder`.
  D7 now defines row 0 as the first cell of the layout string actually applied, in the physical pane order it is built from, and adjusts the pins against that same order; a row-0 pane with no pin gets none.
  The acceptance criterion adds the case of strand panes in a physical order other than the intended one.
- **F2 (MEDIUM, design).**
  D7 now keys the adjustment on the strand window's effective `pane-border-status` read back from tmux, as the reserved rows are keyed on `#{status}`; `off` or a failed readback keeps every pin at its cell height.
  The acceptance criterion adds the `off` readback case.
- **F3 (LOW, design).**
  D7 now picks the floor: a planned one-row top cell has its pin floored at one content row, and that cell ends one row taller, the row coming from the cells below; the "keeps its planned height" sentence names this exception, and the acceptance criterion adds the case.
- **F4 (MEDIUM, decision).**
  D12 now decides that the per-entry revive function, built by the teller from an engine over that worktree's own `reed.yaml`, state and geometry, checks under that worktree's own op lock whether it has a recorded, not-live session, and reports skipped otherwise.
  The booter's `reedengine` only sequences the calls and never reads another worktree's state, so `PATTERN-told-geometry` holds and the check runs under the lock that guards the state it reads.
- **F5 (LOW, consistency).**
  The fused Open-risks line is split; a new bullet names both stops (a live later-position session and a failed predecessor up) instead of pointing at the Bound.
  The D12 acceptance criterion adds the live later-position case.
- **F6 (LOW, design).**
  D8's binding now carries the socket and tmux paths as shell-quoted tokens emitted through `internal/shell` at bind time, and the client name as `#{q:client_name}`, which tmux shell-escapes at key press; the Bound says no runtime value can end a token or reach the shell as syntax.
- **F7 (LOW, consistency).**
  D15 now orders the checks: a content-identical result is reported as unchanged, whatever the permission bits, before the read-only check.
  A failed write, a read-only directory included, is reported as a skip with its reason and removes the temporary file; the Bound covers the temporary file beside the target, which never outlives the run.
  The acceptance criterion adds the read-only-but-unchanged case and checks no temporary file is left after a skip.

## Disputed

None.

## Deferred

None.
