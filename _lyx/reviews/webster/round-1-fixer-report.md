# Fixer-report — webster round 1

## Changes

- F1: `tools/sandbox/SANDBOX-REED-SUITE.md` M19 now says the attached client must show the `shell` button on line 1. A grep of `tools/` found no other `VIEW` label check.
- F2: `internal/battencli/cli.go` package doc drops the content-rows write clause.
- F3 (fixed differently from one detail of the proposal): `Engine.SetContentRows`, `TestSetContentRows`, its doc.go paragraph and its entry in the zoom-bracket call-site list are deleted.
  `zoom_integration_test.go` loses the content-strand report steps and keeps the Selvage-zoomed add.
  `applyOpts.InstallPins`, which only `SetContentRows` set, is removed with its branch in `applyLayoutLockedOpts`.
  The proposed removal of "the `Strand.ContentRows` handling that only it feeds" was not made: the content size kind is a configurable display kind read by `render` and its tests.
  The review allowed that alternative, so doc.go now states that no pane process reports a content line count today and a content strand takes its minimum rows.
  The `internal/reedengine` row in `test-budget.yaml` is lowered by one.
- F4: `applyOpts.SkipWhenBoxEquals`, its guard and doc paragraph in `applyLayoutLockedOpts`, and `TestApplyLayoutLockedOpts_SkipWhenBoxEquals` are deleted.
  The `internal/reedengine` row in `test-budget.yaml` is lowered by one more.

Gate repair: the F3 edit left the `render` import unused in `internal/reedengine/zoom_integration_test.go`, which builds under the `tmux` tag, so the verify gate failed to compile it.
The import is removed.
Earlier `go vet` runs used the `integration` tag only and missed this file.

Verified with `lyx gate test ./internal/reedengine/... ./cmd/lyx` (green) and `go vet` on `internal/reedengine` with no tag, with `tmux`, and with `integration tmux`.
The tmux-tier and integration tests were not run.

## Disputed

None.

## Deferred

None.
