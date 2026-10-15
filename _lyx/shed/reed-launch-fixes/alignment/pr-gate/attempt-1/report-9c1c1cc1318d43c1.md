---
verdict: APPROVED
---

## Alignment at pr-gate: reed-launch-fixes

Measure: board entry `reed-launch-fixes` (issues #627, #628).
The decision record carries no `Added after Discussion` entries, so the board body is the measure unamended.

### Scope against the board entry

- Part 1 (#627), concurrent launches corrupt the launch line: `launchStrandLocked` in `internal/reedengine/spawn.go` now sends the dialect's discard-line key ahead of the literal launch payload, and `internal/shell` gains `DiscardLineKey` (`C-u` on POSIX, empty on PowerShell).
- Part 2 (#628), `--focus=false` still takes focus: `render.Display` gains `KeepViewer`, `lyx shuttle run` sets it to `!focus`, and `newAgentWindowSelected` returns false for a display that sets only `KeepViewer`, so the new window is never selected.
- The entry's "also seen" items, a flag to open the window outside the caller's session and the `--anchor hidden` startup failure, are in scope only if the fix needs them; the diff leaves both alone, which the entry allows.
- `internal/reedengine/doc.go` names the new flag and the discard key in the same commit.
- No change goes beyond the two parts: the 13 changed files are reedengine, its render types, the shell dialects, shuttlecli and their tests.

### Open risks carried in the decision record

- Stray input can still reach a booting pane when a `Focus` or follow-rule window takes the viewer, if it arrives after the `C-u`.
- PowerShell panes get no discard key, so the launch-line fix is POSIX-only.

Neither names an unrecorded parent correction.
The reported failure was `--focus=false` launches, which no longer switch the viewer onto the booting pane, so both risks are residual limits rather than a missed point of the entry.

### Overlap with peers and parent commits

- `shed-tackler` (row 7, Plan-Write): no changed paths yet; its scope is shed's driver and tackler, with no overlap with reed, shell or shuttlecli.
- `twill-fork0` (row 16, Describe): changes stencils, burlerengine, loomengine, twilldirective and `docs/overview.md`; no file or behaviour shared with this run.
- Parent commits: none listed.

No overlap, so no landing order applies; this run is at row 22, ahead of both peers.

### Board entries touching this task

- `run-pane-timing` depends on `reed-launch-fixes` and stays open: it asks for status-pane timing, which this run does not deliver.
- No board entry is made obsolete by this run.
