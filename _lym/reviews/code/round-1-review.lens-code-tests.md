```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — per behaviour the diff adds or changes, the test that pins it, or none.

**Per-behaviour list**

1. `buttonRunDirectory` returns the worktree's current anchor, read afresh (`internal/reedcli/button.go`): pinned by `TestButtonRunDirectory` rows `SubpathMarkerRecordedAfterARootRenderRunsInTheSubpath` and `RootMarkerRecordedAfterASubpathRenderRunsAtTheRoot`. Both would fail if the anchor were taken from the rendered directory.
2. `buttonRunDirectory` with no marker runs at the worktree root: `NoMarkerRunsAtTheWorktreeRoot`.
3. `buttonRunDirectory` when the render already matches the marker: `RenderMatchingTheMarkerRunsWhereItWasRendered`.
4. `buttonRunDirectory` for a directory in no worktree returns `ErrNotAGitRepo`: `DirectoryInNoWorktreeIsAnError`.
5. `runButtonVia` runs a plain entry in `runDir`, not `spec.Dir`: `AnchorMovedAfterTheRenderRunsInTheCurrentAnchor`. It asserts the run call's dir is `runDir`, which differs from `dir`.
6. `runButtonVia` opens the confirm prompt's line in `runDir`: `AnchorMovedAfterTheRenderPromptsWithALineOpeningInTheCurrentAnchor`. `wantConfirm(runDir)` builds the expected line.
7. `runButtonVia` with an unresolvable anchor runs nothing and shows "the anchor of DIR does not resolve: …": `UnresolvableAnchorRunsNothing`. The row asserts no run, no confirm and the message.
8. `runButtonVia` passes the rendered `spec.Dir` to `anchorOf`: the `t.Errorf` inside the `anchorOf` stub in `TestRunButtonVia` covers every row that reaches it.
9. The existence check still runs on `spec.Dir` before `anchorOf`: the `RemovedDirectoryOf…` rows. `anchorOf` is not reached there, since the stub's `t.Errorf` would fire only for a wrong argument, and the message assertion pins the "is gone" text.
10. The `RunButton` signature change and the `anchorOf` field in `buttonDeps`: the compiler pins them. `buttonclick_integration_test.go` is updated with an identity `anchorOf`.
11. The Info log when `runDir != spec.Dir`, and the Warn log on an anchor error: none. These are log lines only, and the repo's other log lines are not asserted either, so this is not a finding.
12. The `could not raise its prompt; see the lym log in runDir/.lym/logs` message now uses `runDir`: no row sets both `exeErr` and `anchorMoved`, so a change back to `spec.Dir` there would pass. This is the only gap I found.
13. The confirmed re-entry (`req.Confirmed`) resolving the anchor again: the `anchorOf` stub runs on every row, including the existing confirmed rows. A row where the anchor moved between the prompt and the confirmed pass does not exist. The behaviour is the same code path as 5, so this is not a gap.

**Findings**

- Severity: LOW. Class: scope. Location: `internal/reedengine/buttonclick.go:126`, test table in `internal/reedengine/buttonclick_test.go`.
  Summary: the prompt-failure message now names `runDir/.lym/logs`, but no row combines `exeErr` with `anchorMoved`, so a regression to `spec.Dir` there passes.
  Evidence: the added rows at `buttonclick_test.go` (the three `Anchor…` rows) leave `exeErr` unset. The existing `exeErr` row uses an unmoved anchor, where `runDir == dir`.
  Fix: add an `exeErr` plus `anchorMoved` variant row, or an `anchorMoved` field on the existing `exeErr` row.

No other behaviour lacks a pinning row. I found no redundant tests: the `button_test.go` rows each cover a distinct marker and render combination.

**Tool calls:** none. My context already held the diff and the definitions.
