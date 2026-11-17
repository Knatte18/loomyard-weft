```yaml
lens: fix-code-correctness
outcome: reported
```

**Lens fix-code-correctness: two findings, both MEDIUM**

**1. MEDIUM. `RemoveHandedOverPair` can now delete and push removal of a record another machine owns.**
- Changed text: `internal/pairteardown/leftover.go`, `RemoveHandedOverPair`, which now calls `td.Run` where it used to call `td.RemovePair`.
- `Run` ends with `removeCancelledRunRecord`, and that runs for every request, `Local` included (`teardown.go:565`).
- When the local batten record reads `cancelled`, `removeCancelledRunRecord` removes the run directory (`followup.go:91`). It then commits the removal on the prime's records side and spawns a push (`followup.go:101-108`).
- `leftoverKind` returns `LeftoverHandedOver` for any run with `HandOverMarked && OwnedElsewhere` (`leftover.go:75`) and does not look at the run's state.
- Concrete input: this machine handed over a run whose local record is `cancelled`. The hand-over mark is written and the pair is still standing. The dobby calls `RemoveHandedOverPair`, and `Run` deletes the record and pushes the deletion. The run now belongs to another machine, so that deletion can reach that machine's record on origin.
- The old code called `RemovePair` directly and never reached this step.
- Possible fix: put the session end and removal back as two calls, `td.EndSession` then `td.RemovePair`, so the cancelled-record cleanup and the follow-up warning are not reached.

**2. MEDIUM. `MarkKeptCancels` now fails on every already-committed kept cancel while a merge is in progress.**
- Changed text: `internal/battencli/leftoverpairs.go`, the new `else if` branch that sends a finished keep intent naming this machine on to `commitCancelRecord`.
- `commitCancelRecord` calls `refuseWhileMergeActive` first. Take a kept cancel that is already committed, cancelled with keep by this machine, with the pair standing, and a merge in progress on the prime's records side.
- Before the fix, such a run was skipped at the `Cancel != nil` guard. Now `refuseWhileMergeActive` returns an error, so `MarkKeptCancels` stops with "commit and push the kept cancel of <slug>" even though there is nothing to commit.
- The dobby then reports a failing marking and holds the kept notice until the merge ends. It can also hide later runs, because the loop stops at the first error.
- Possible fix: for a run that already holds an intent, make the merge refusal a skip that is only logged, or skip the commit when the status file is unchanged.

**Tests**
- The F2 notice-failure test, the F10 pair-skip test and the F8 rows would fail with their defects in place.
- The F15 test is a coverage addition and passes with or without a defect, as the review intended.
- No caller of a changed signature is left broken: `tearDownLeftover` discards `RemoveHandedOverPair`'s first return value, and `dueOrphans` keeps its signature.

**Tool calls**
1. `grep` for `RemoveHandedOverPair` callers, the `removeCancelledRecord` references in `teardown.go`, and the `leftoverKind` source. My context lacked the callers and the `leftoverKind` body.
2. `grep` for the `removeCancelledRunRecord` source in `followup.go`. My context lacked what `Run` does after the removal.
