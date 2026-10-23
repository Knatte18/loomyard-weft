---
verdict: APPROVED
---

## Alignment at pr-gate — orch-compact-repeats

Measured the diff of PR 846 at `de55116e` against `main` against board entry `orch-compact-repeats`, as amended by the decision record's one `Added after Discussion` entry by the parent.

### Against the amended entry

- The entry's expectation, that the watcher never types another `/compact` for the same request once a compact has landed, is met in `internal/orchengine/watcher.go`.
  `withdrawQueuedCompact` reads the pending `/compact`'s outcome again after the withdraw.
  An inject still queued or sending keeps the step pending and the boundary unconsumed through `errCompactInFlight`, and the next tick withdraws again.
  The change also updates the compact-cycle paragraph of `internal/orchengine/doc.go`, the `ResumeLineWithdrawn` comment in `state.go`, the creel fake and the compact tests.
- The parent's added decision, switching the reed pane-tail separator from `\x1f` to a tab in its own commit, is met by `de55116e`.
  The commit changes `paneTailSeparator` in `parse.go`, both `-F` formats in `overlay.go` and `ownwindow.go`, the parse comment, the format line in `internal/reedengine/doc.go` and the reedengine tests that build listing lines.
- The handoff-timeout drop of a step that stays in flight is not in the decision record.
  It bounds a wait inside the compacting phase the record scopes in, so it is a choice within the writer's design, not a scope deviation.
- Nothing outside the amended entry is changed: creel's delivery worker, shuttle's send path and the reload step order are untouched, and the diff makes no `PATTERN.md` or `docs/` change.

### Open risks

The record carries one open risk: a delivery attempt that types the `/compact` while the watcher waits still compacts the session a second time.
It names a bound only creel could close, not an unrecorded parent correction, so it does not escalate.

### Overlap with peers and parent commits

`lyx loom peers` lists no parent commits and three peers, none of which touch `internal/orchengine` or `internal/reedengine`:

- `batten-cancel-verb` (row 11): `internal/fabricengine`, `internal/proc`, `internal/shedrun`; no overlap.
- `plan-card-rules` (row 11): plan parser, loomshed, pattern, batcher and `tools/plandag`; no overlap.
- `prime-owner-visible` (row 20): `internal/boardcli`, `internal/boardengine` and the board skill; no overlap.

With no overlap, no landing order applies; this run sits at row 21, ahead of all three.

### Superseded

No other board entry covers the watcher's repeated `/compact` or the pane-tail separator, so this run supersedes none.
