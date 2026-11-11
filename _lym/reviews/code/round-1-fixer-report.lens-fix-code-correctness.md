```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed code and the added tests through the fix-code-correctness lens.

No findings. I traced each changed or added function and test by hand against the code in my context and found no wrong result, broken caller or vacuous test.

- **`CheckAddedEntriesSubset`**: the per-text counter is decremented on each match. With current holding one entry, a next that repeats it matches the first copy and reports the second. This is the new "the file repeats an entry" row. Current holding two and next holding two gives no finding.
- **`recordHistoryFrom`**: `recordHistory` passes the same arguments as before, and `readVersions` is still read only when a trailer names the record. In the test, removing the `readWhole` fallback returns an empty history. Then `len(history) < len(whole)` is true while `wantCut` is false, so the uncut rows fail as they should.
- **`boardRecordStoredOf`**: `record` at the call site is the `runRecord` from `runRecordOf`, not the `[]byte` parameter of the `storeRecord` closure. The test's `errors.Is(err, nil)` is true only when `err` is nil, so it is correct.
  - Dropping the `project` branch makes the `BeamProjectRecordWhoseEntryHoldsOne` row return true and fail.
- **Fake changes**: `committedAfterErr` now also triggers once `replaceCommits > 0`. No `add` test sets `replaceCommits`, so none changes.
  - `recordWrites` counts only `writeRecord` calls. The setup in `newReplaceFake` writes with `os.WriteFile` directly, so the first call counted is the replace's own write. Restore is the second.
- **Replace refusal rows**: `RecordWriteRunRecordNotCommitted` sets `recordRow`, so `checkDarnRecord` runs and, at row `Drift-Baseline`, returns the `goto` way forward.
  - Without that call the flow would hit the later "not committed" refusal. That one names `lym fabric commit` and lacks the `goto` fragment, so the row fails.
  - `RecordFileAbsent` fails if the not-exist branch is removed, because the message would read "read the decision record" instead.
- **Commit-outcome rows**:
  - The unreadable-HEAD row works because `commitReplace` increments `replaceCommits` before `landReplace` re-reads HEAD, so the read returns `committedAfterErr`.
  - The `headCarriesErr` row needs `replaceErrLands`, which it sets, to get past the bytes check.
  - The restore-failure row fails on the second write, leaving the folded record in place, as asserted.
- **Store lock test**: removing the lock from the store verb leaves `lockCalls` at 0 in the success row. In the lock-error row the store would succeed instead of refusing. Either way the test fails.

One weak spot that is not a finding: `cut := len(history) < len(whole)` cannot detect a cut that lands on the oldest version. No current row does that.

Tool calls made: none. The context already held every changed file.
