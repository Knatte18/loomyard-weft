# Fixer report: discussion, round 1

Every finding was checked against the code, and every premise held.
All seven are fixed in `_lyx/discussion/decision-record.md`, with matching edits in `support-log.md`.
`lyx loom validate-discussion` passes on the result.

## Fixed

- **F1 (BLOCKING, design), D3's rejection bound.**
  Confirmed in `internal/shedengine/run.go`: `episodeStuckCount` stops at a Done by any `segmentEnders` member, and `validate.go` requires `on_stuck` to share the segment, so a `Darn` Done resets `PR-Gate`'s count.
  D3 now says that each traversal of the edge is one operator `lyx loom reject`, that this is the edge's whole bound, that `max_bounces: 5` does not cap it, and that each traversal is one `Darn` spawn with its own verify budget.
  The rationale adds that a human starts every traversal.
- **F2 (BLOCKING, design), D10's clearing condition.**
  Confirmed: `markTaskDone` in `internal/landingshed/finalize.go` only warns, batten's `Worktree-Teardown` follows only a done child, and `RunLockedError` makes clearing depend on batten's state.
  D10 now settles the claim by run outcome.
  A landed teardown, flagged by batten's row, marks the entry done.
  On `lyx fabric remove`, the entry is left alone with a warning while the slug's batten run is `running`.
  It is marked done when that run is `done`, and cleared when the slug has no batten seed or its run is halted.
  D10 now says what the clearing lets through and what bounds it.
  The Goal, the Scope line, the D10 title, the Open risks, an Auto-mode assumption and the acceptance criteria were updated to match.
  The support log's Interview answer, ledger row 11 and a rejected alternative (clearing on board state alone) were updated too.
- **F3 (BLOCKING, consistency), side vocabulary in D9 and D11.**
  Confirmed against `pattern/PATTERN-fabric-vocabulary.md`: the owner set excludes `landingshed` and `loomcli`, and owner-set exports are no exception.
  D9's Stuck now names the parent pair, with `lyx fabric commit` run in the parent pair, and every text `landingshed` produces for the step avoids side names.
  D11 now lists each dirty path as an absolute path with no side label.
  The Constraints entry is now `PATTERN-fabric-vocabulary` and covers `loomcli`.
- **F4 (MEDIUM, design), empty verify guarded only at start.**
  Confirmed: `landingshed/verifygate.go:158` skips with a warning when the command is `""`, and `VerifyCommand` already returns `(string, error)`.
  D6 now says darn's reader returns an error for a missing file or an empty `verify`, so the gate, Publish and Finalize stop at call time.
  D7 calls the start refusal the early check.
  A new acceptance criterion covers an empty `verify` after start.
- **F5 (MEDIUM, decision), the rejection record's lifecycle.**
  Confirmed: `PR-Rework` clears the record through `ClearRejection` after its round commit, and `rejectionPending` gates `statusCommitPathspec`.
  D4 now says the `Darn` row removes the record when a spawn that rendered it ends Done, after the landing commit, and a removal failure is a returned error.
  A gate-failed Stuck keeps the record.
  A new acceptance criterion covers this.
- **F6 (LOW, design), verify budget per spawn and log retention.**
  Confirmed: `run.gateFails` in `internal/shuttleengine/wait.go` is per shuttle run.
  D4 now says the budget is per spawn, resets on a rejection re-spawn and on a resume, and that each spawn is started by an operator act.
  D8's rationale now justifies twenty by the failure record's pin, and no longer claims twenty covers a whole run.
  The `Darn` acceptance criterion now says the budget is counted per spawn.
- **F7 (LOW, consistency), untraced choices.**
  A new acceptance criterion covers D12's orch stencil, the `ly:board` skill and the board `recipe` help.
  The failure-record criterion now also covers the `tmux`-tier confirmation for a `publish_verify` failure.

## Disputed

None.

## Deferred

None.
