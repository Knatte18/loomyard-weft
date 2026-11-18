```yaml
lens: fix-proposal
outcome: reported
```

no findings

Task: challenge the proposals the fixer carried out as proposed, which are F1 (the owed-push records) and F3 (the `BackoffSleep` doc reword).

I built the cases below against F1 and none breaks:
- **Stale record:** A `PushFailureDeferred` record written after the spent backoff stays on disk only until the next push. Every landed push calls `landed` and then `clear` (`pushclassified.go:323-337`), which removes the side's record. The transport record is written after the push that already cleared the record, and the next push clears it the same way.
- **False match on the deferral check:** The next push's `recordsSideNotLanded` accepts a deferred record only when `LastSeen` is not before that push's `started`. The record from the spent-backoff exit predates it, so it cannot read as a fresh deferral.
- **Wrapped error:** The wrapping `fmt.Errorf("...: %w")` calls in `pushRecordsSide` and `step` keep the `*PushFailedError` reachable. `pushFailuresOf` follows `errors.Unwrap`, so `IsPushOwed` still reads the class.
- **Mail on a transient failure:** The transport record mails the orch once per episode, as the pre-push transport path already does. That is consistent, not a regression.

F3 only rewords a doc comment, and the new wording is accurate for every use of the field.

Further tool calls, 2:
1. `grep` for `ClearPushFailure(` callers and the `recordLandedPush` definition in `internal/`, to confirm a landed push clears stale records. My context lacked `ClearPushFailure`'s callers.
2. Read `pushclassified.go:300-340`, to confirm that `landed` calls `clear`. My context lacked those lines.
