```yaml
lens: fix-affected-set
outcome: reported
```

Task: check, through the affected-set lens, that the F1–F3 fixes cover every member of each finding's set.

No findings.

- **F2 (callers of `checklistPoll`):** the only callers are `checklist.go:127`, which passes `reviewWaitingFor(location)`, and the new test at `checklist_test.go:1287`. No caller is left on the old one-argument signature.
- **F1 (stubs with `Steady` equal to `Text`):** outside `reviewstatus_test.go`, no other `WaitNote` stub in a `statusCmd` test gives the two fields the same value. The remaining one, `status_test.go:787`, is the hook-error case, where the note is dropped, so it can't tell `Text` from `Steady`. I did not check the `Steady: note` stubs in `reviewstatus_test.go`, such as `fixedNextStep`. They test hook-chain composition, not the one-shot status or pane path.
- **F3 (the unavailable key):** `statusWatchLine` has one unavailable branch, and the new assertion covers it.

Calls made:
1. `grep checklistPoll(` over all Go files, to find every caller of the changed signature.
2. `grep Steady:` in `_test.go` files under `internal`, to find stubs where `Text` and `Steady` are equal. My context didn't show tests outside the files the fixer read.
