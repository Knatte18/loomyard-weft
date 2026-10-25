```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals of F1, F2, F3 and F4 that the fixer carried out as proposed.

No finding of BLOCKING or MEDIUM severity. Two LOW findings:

1. **LOW**: `internal/dobbyengine/passes.go`, the new `delete(d.liftPending, run.Slug)` in `drivePasses`.
   - The review's F3 fix says to keep `liftPending` while a pass is in flight. The delete is guarded only by `run.State != runStateRunning || run.CancelMarker`, not by `!trigger.InFlight`.
   - Case: a lift is seen while a pass is in flight. The run's state or cancel marker then flips mid-pass, so the delete fires and the in-flight pass's lift is dropped.
   - Impact is small, because a non-running or cancelled run gets no pass anyway. The code still departs from the review's stated proviso.
   - Evidence: the `triggerFor` result carries `InFlight`, and the delete ignores it.

2. **LOW**: `internal/dobbyengine/passes_test.go`, the old `CoveredLiftGivesFull` row (replaced).
   - The `covered && trigger.Lift` branch in `planPass` is still in the code, but no row pins it now.
   - It stays reachable. `liftPending` is kept while a pass is in flight, so a run that is re-covered before the pass exits plans with `covered=true` and `Lift` set.
   - The review told the fixer to change that row to `covered=false` and keep the `OwnedElsewhere` lift rows. Only `OwnedElsewhereCoveredLiftGivesFull` still covers the covered case, and that row takes the earlier owned-elsewhere branch. The ordinary covered-lift branch is unpinned.

F2 and F4 held up under challenge:
- **F2:** `migrateDraftStatus` now runs before `loaded` is cloned and before `fn`.
  - A null-status release (before `draft`, after nil) is skipped by `applyDraftRule` and stays released.
  - An import, a rename or a `noWrite` result gives the same outcome as before.
  - I found no case where `fn` reading `draft` instead of an empty status breaks anything.
- **F4:** the comment reword is accurate against the review's description.

Tool calls: none; my context held everything the lens needed.
