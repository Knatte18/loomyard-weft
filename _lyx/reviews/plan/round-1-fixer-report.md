# Plan-Review round 1 — fixer report

All four findings fixed.
`lyx loom validate-plan` passes after the edits (informational `create-new-unit` findings only, unchanged from before).

## Changes

- **F1 (MEDIUM)** — `_lyx/plan/04-send-launch-script-source.md`:
  added `internal/reedengine#Engine.ReplaceStrand` to the Edit list;
  added a Failed-add bullet: `ReplaceStrand` tracks a post-add `SaveState` failure apart from a launch failure, and deletes the new strand's script only when that save failed and `reconcileApplyPersistLocked` also errors (the record was then never persisted);
  added a `ReplaceStrand` failing-save test;
  widened the `ImpactSummary` to name the unpersisted-record delete.
- **F2 (MEDIUM)** — same card: added an `AddStrand` test whose `Enter` send-keys hook replaces `reed.json` with a non-empty directory, so the post-add `SaveState` fails after the script write, asserting the launch directory is left empty.
- **F3 (LOW)** — same card: added `internal/reedengine/emptycmd_integration_test.go` to the Edit list, with an Intent sentence rewording its file doc so the prelude alone is the script's content and the send-keys literal is its source statement.
- **F4 (NIT)** — `_lyx/plan/03-launch-script-cleanup.md`: the `ReplaceStrand` test's replacement spec is `render.AnchorHidden`, so the launch the default fake would fail is skipped and the test isolates the removal step.

## Deferred

None.
