```yaml
lens: fix-proposal
outcome: reported
```

No fix-proposal findings that break anything; one LOW doc gap.

- **F1, LOW.** Changed text: the extra conditions on the hold branch at `starts.go:89` (`!task.OwnedElsewhere && len(task.PendingDependencies) == 0 && !covered`). The proposal holds. A task with a `StartRefusal` that is also owned elsewhere, has open dependencies or sits under a pause now falls through to `readyToStart`. That is false through `heldByEntry`, so the task is skipped with no spawn and no notice, as before the diff. `BuildView` still shows its `StartRefusal` (`view.go:100`), so `lym beam status` loses nothing. The gap is `doc.go:97-99`. It says dobby records the refusal and posts the notice for a released task with a finished run record, and it does not say the hold applies only when no other wait covers the task. A reader would expect a notice in the owned-elsewhere case. Suggested change: add "unless the task is owned elsewhere, has open dependencies or is covered by a pause" to that bullet.
- **F2.** No finding. The new help sentence matches what `overviewdobby.go` draws. The `battencli/cli.go:27` doc comment says the same: a ready task shows the last refused start, and the waits list includes a start batten refuses.
- **F3.** No finding. The new row fails with the defect in place. If `heldByEntry` dropped `StartRefusal`, `finished` would pass the `!heldByEntry` check at `view.go:107` and take the one slot (`slots` 1 to 0). `ready` would then get `AtMaxRuns: true`, and the row's want has none.

Tool calls:
1. Read `view.go` lines 80-125, to check whether the F3 row fails with the defect in place. My context did not show `BuildView`'s slot accounting.
