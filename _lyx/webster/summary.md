# Reed window order: overview first, agents in start order, shell last

Both batches recorded `done`, so `outcome: done` with two batches done.

## Batch 01-selfexec-helper (head `56dc846cc3618eb267c51b356bc42f3d04b0558e`)

Covers cards 1–5: the `internal/selfexec` re-exec leaf, the redrawn loom checklist, the plain overview panes that re-exec on a changed binary, the watchdog re-apply skip for the last applied or last failed box, and the window order around a `shell` window.
The fork reported two additions beyond the card text.
It fixed `TestWatchLoop_NonSentinelFailureDoesNotGoDormant`, which the new skip rule broke.
It also left `applyOpts.SkipWhenBoxEquals` in `apply.go` with no production caller.
A skip at the failed box still logs the "recovered" Info line.
The `tmux` tier was compiled only, not run.

Scope guard deviations (files changed outside the batch's own targets):
`internal/battencli/overview.go`, `internal/battencli/overview_test.go`, `internal/loomcli/checklist.go`, `internal/loomcli/checklist_test.go`, `internal/reedengine/attach.go`, `internal/reedengine/reapply.go`, `internal/reedengine/watchloop_test.go`, `internal/reedengine/windoworder.go`, `internal/selfexec/leaf_enforcement_test.go`, `internal/selfexec/selfexec.go`, `internal/selfexec/selfexec_test.go`, `internal/statuspane/pane.go`.

## Batch 06-overview-anchor (head `4514e4fcac4e0e3b418d2ae08c220325d1462c3c`)

Covers cards 6–9: the `overview` anchor, the focus rules, Alt+Up and Alt+Down window stepping with the `shell` button, and the move of the status, runs, driver and orchestrator strands onto the new placements.
The fork reported these differences from the card text:
- Card 7: `killStrandPanes` calls a new `selectRelevantWindowLocked`, and two now-unused select helpers were deleted.
- Card 8: a new `hiddenWindowCondition` constant in `bindings.go`.
- Doc changes beyond the card lists: `internal/loomcli/doc.go`, `internal/reedengine/doc.go`, `docs/overview.md`, `internal/orchengine/doc.go`, `internal/shuttleengine/doc.go`, and a comment in `internal/shuttleengine/spec.go`.
- The `internal/reedengine` test budget went from 265 to 266.

The `tmux` tier was compiled only, not run.

Scope guard deviations:
`internal/orchcli/start.go`, `internal/reedengine/attach.go`, `internal/reedengine/attach_test.go`, `internal/reedengine/bindings.go`, `internal/reedengine/ownwindow.go`, `internal/reedengine/render/policy.go`, `internal/reedengine/windoworder.go`.

## Verify gate

Not run by Master; the gate runs the plan's `## verify:` after this turn ends.
