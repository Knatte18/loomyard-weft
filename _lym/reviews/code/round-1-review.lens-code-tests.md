```yaml
lens: code-tests
outcome: reported
```

**Per behaviour: the test that pins it**

1. `refuseAtStart` offers a done status to `ClearMotifRun` on `run` and `step` but not `goto`, and does not offer a running run.
   - Pinned by `TestRefuseAtStart_ClearsADoneMotifRunBeforeJudgingTheStart` (`startguard_test.go:440`), rows `RunOver…`, `StepOver…`, `ARunningRunIsNotOffered` and `GotoIsNotOffered`.
2. A cleared run is judged as a fresh start, and a done run left in place is not.
   - Pinned by the same test, rows `RunOverAClearedRunIsAFreshStart` and `ADoneRunLeftInPlaceIsNoFreshStart`. It sees the fresh start because the `Raddle` seam is only called when `!found`.
3. A failed clear refuses, wraps the error and names the `lym batten run <slug>` re-run.
   - Pinned by the same test, row `AFailedClearRefuses`.
4. `startClearsMotifRun` as dobby reads it in `DobbyTasks`: a done motif run of a raddle entry holds no start, a done raddle run of a raddle entry keeps it, and a done motif run of a darn entry keeps it.
   - Pinned by `TestDobbyTasks_ReadsTheStartRefusalOfAFinishedRunRecord` (`dobbyreading_test.go`), the three new rows.
5. `finalizeEnteredRead` exempts a pair status standing at Raddle-File.
   - Pinned by the new `TestReconcileOrigin_Ended` subtest and by the new `TestRecordsReplay` row "A pair status at Raddle-File is not set aside". They cover the reconcile and the records-replay readers.
6. A Raddle-File history row alone does not exempt a run, so an origin that really ended the run still reconciles as ended.
   - Pinned by the subtest "A raddle run a return sent back to Raddle-Write is ended".
7. `sketchFinish` mails the parent on a refused Sketch, with the dependent entry and both ways forward, and mails nothing when the board takes the Sketch.
   - Pinned by `TestSketchFinish_TellsTheParentOnlyOfARefusedSketch` (`commitstatus_integration_test.go:107`).
8. The draft fallback stays: the entry is left draft after a refusal.
   - Pinned by the existing row `RefusedSketchLeavesTheEntryDraft` (`commitstatus_integration_test.go:53`).
9. `clearMotifRunOfRaddleEntry` (`startguard.go:146`), the closure wired into `hubStartGuard`. Its paths are a board read error, a slug absent from the board, an entry of another recipe, a raddle entry whose child ran another recipe, and a raddle entry with a motif child that is cleared through `ClearMotifRun`.
   - **No test.** The `refuseAtStart` test fakes the seam, and the `DobbyTasks` test only reaches `startClearsMotifRun`.
10. `DobbyTasks` when the child-recipe read fails: it warns and the run holds the start (`dobbyreads.go:327`).
    - **No test.**
11. `sketchFinish` when the notice cannot be sent: it warns and `Finish` returns nil.
    - **No test.** `notify` is injected, so a row with a failing `notify` would be cheap.
12. `sketchFinish` when `SetRunStatus` fails: it returns the error and posts no notice.
    - **No test.**
13. A run with no parent: `NotifyParent` returns `posted=false` with a nil error.
    - Not at `sketchFinish` level, but it is `orchcli`'s own behaviour and not new in this diff, so no finding.
14. The `ClearMotifRun` records-commit message changed to `batten: …`.
    - No test asserts it (`TestClearMotifRun_RemovesOnlyADoneMotifRun` asserts paths only). The message is not behaviour, so no finding.

**Findings**

- **MEDIUM, `design`, `internal/battencli/startguard.go:146`:** `clearMotifRunOfRaddleEntry` has no test.
  - It is the only code that decides, at batten's start, whether the board holds the slug as a raddle entry before calling `ClearMotifRun`.
  - Uncovered: the board read error, the absent slug and the other-recipe entry. The raddle-entry path through `ReadChildRecipe` to `ClearMotifRun` is also uncovered.
  - Evidence: `startguard_test.go:440-501` fakes the `ClearMotifRun` seam, and `dobbyreading_test.go:194-215` only reaches `startClearsMotifRun`.
  - Fix: a tier0 table-driven test over a fixture board and a fixture run directory.
- **LOW, `design`, `internal/loomcli/wiring.go:249`:** the failed-notice path has no test.
  - The new test's `notify` always returns nil, so a regression that returned the notify error would hold the run's finish and the next transition would retry the whole Sketch write.
  - Fix: one row with a failing `notify` that expects `Finish() == nil`.
- **LOW, `design`, `internal/battencli/dobbyreads.go:327`:** an unreadable child recipe in `DobbyTasks` (warn, run holds the start) has no row.
- **LOW, `design`, `internal/loomcli/wiring.go:246`:** a failing `SetRunStatus` returns the error without posting a notice, and no test covers it.

Outside this lens, for the handler: `startClearsMotifRun` reads `task.Recipe` directly, while `readRaddleStartRefusal` (`startguard.go:179`) resolves the recipe through `boardRecipe(board, entry)`. An entry that gets raddle by that resolution rather than by its own `recipe` field would not be cleared, and no test shows which way it goes.

Tool calls: 1. I grepped the loomcli and boardengine tests for the draft fallback and Sketch refusal coverage, because my context did not show whether `RefusedSketchLeavesTheEntryDraft` already covered the fallback.
