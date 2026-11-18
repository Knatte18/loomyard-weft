```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Searches I ran, one per set-premise finding, plus the ones I judged covered from context:
- **F5 (wording of the stop error):** the grep for the old text, "stop the step's runs", over `*.go` and `*.md` outside `_lym` and `.lym` found no site left. The earlier repo-wide grep found only `looparm.go:122` and `looparm_test.go:114`, and both were changed.
- **F6 (the "names none" doc line):** the same grep shows `names none` in other files, but all of them are unrelated sentences, not copies of the `StopOrphanedStepRuns` doc line.
- **F4 (the re-step wording in the reconcile paragraph):** the grep for `leave.*runs to the re-step` finds no copy in any other file. The paragraph in `internal/shedverbs/doc.go` was the only site.
- **F3 (hooks table):** `looparm_test.go` has one table of hooks over the stop-note renderer. The new `OrphanedSteps` row sits in it, so every report row and the failing-stop row now run through `orphanedStepsNote`.
- **F1 (re-check before the stop):**
  - **Covered from context:** the re-check sits in `stopStepRunsWhere`, which both `StopStepRuns` and `StopOrphanedStepRuns` use. For `StopStepRuns`, `selects` is a tie comparison, which the re-check satisfies for a run that still names the tie. The new `attached` row covers the orphan form.
  - **Not checked:** no repo-wide search for other callers of `stopStepRunsWhere` or `loadRunState` patterns.
- **F2 and F7:** they concern one test, and both are fixed in it. The other rows seeded with PID -1 (`foreign`, `untied`, `terminal`) are not named gone, so they need no change.
- **Unchecked:** the `_lym/landing/summary.md` sentence from F1 is outside the fixer's write surface, so I did not search it.

Tool calls:
1. `grep -rn` over `*.go` and `*.md` for the F4, F5 and F6 phrases, to confirm no copy remains. My context held only a grep of the F5 text from the earlier pass.
