```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed code (F1–F4) by running concrete inputs through it.

No findings.

What I traced:
- **F1, `planPass`:** with `covered=false`, `Lift` set, no owed pass and the run at Run-Shed, the new `|| trigger.Lift` returns `full`. It returns `passFullNoReopen` when `OriginOwed` is set. The `OwnedElsewhere` and covered branches are unchanged. The `LiftGivesFull` rows fail without the tail change, because the old code returned `passContinue`.
- **F1, existing in-flight test:** the second spawn is now a full pass, so the added `continuePass` assertion holds.
- **F3, `liftPending` cleanup:** the delete runs after `triggerFor` and `planPass`, so a lift seen on a non-running or cancel-marked run is dropped in the same tick. `wasCovered` is already false on the next tick, so the lift is not re-set. A held driver lock reads as in flight and does not trigger the delete, which matches the fix. The unlisted-slug sweep is correct.
- **F3, new test:** the blocked tick sets then drops the lift, and the next probe tick gives a continue pass. With the defect in place that pass would be full, so the test fails without the fix.
- **F2, `boardCriticalSection`:** migrating before the `loaded` clone means a null-status release sees `before=draft` and `after=nil`, so `applyDraftRule` skips it and the task stays released. A `noWrite` result or an error from `fn` leaves the migration in memory only. The new test fails without the fix: `SetStatus` always writes, since `store.SetStatus` returns nil with no `noWrite` short-circuit. Under the old order the migration would re-draft the task.
- **F4:** comment-only change, no callers affected.

Residual risk, not a defect: `fn` closures now see `draft` instead of an empty status on the first write to an unmigrated board. I did not read the other `fn` bodies, such as the start condition, to see whether any depended on the empty value.

Tool calls (1 of 6):
1. `grep` of `SetStatus` in `internal/boardengine/board.go`. I needed to know whether it short-circuits to `noWrite` on an unchanged status, which would let the F2 test pass with the defect in place. It does not.
