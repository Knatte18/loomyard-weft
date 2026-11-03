```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens over the F1–F4 change set (commit c0e00afa).

**Findings**

1. **MEDIUM — the token soft-fail rows pass with the `tokenFailure` guard removed.**
   - **Changed text:** the new rows `AnUnresolvableTokenOnTheParentRuleReadReadsAsNoRuleAndIsDone` in `prreview_test.go` and `AnUnresolvableTokenOnTheParentRuleReadReadsAsNoRuleAndLandsLocally` in `finalize_pr_test.go`.
   - **Problem:** both script a bare `githubclient.ErrTokenUnresolvable`. F3 exists to pin the `!tokenFailure(err)` guard in `resolveWithParentRule` (`pullrequestmode.go:117`).
   - **Evidence:** `githubclient.IsTransient` (`transient.go:21-45`) returns false for a bare sentinel, because the sentinel is not a `*github.ErrorResponse` or a `*url.Error`. Take the guard away and the run is:
     - `transientFailure` returns nil.
     - `recordFound` is false in both rows.
     - The Warn path runs, giving Done.
     - Both rows still pass.
   - **Contrast:** the existing token rows (`prreview_test.go:256`, `finalize_pr_test.go:181`) wrap the sentinel in `&url.Error{Op: "Get", ...}` for this reason. The scripted error needs the same wrapper to be classified transient. Without the guard it then becomes an error stop, and the row fails.

2. **LOW — the new `ReviewEnded` open case in `judge` returns Done without checking the head is gated.**
   - **Changed text:** `prreview.go:685-690`.
   - **Problem:** with a clean tree, a head that failed its gate (mailed, not gated) still ends Done. Finalize then merges that head on GitHub.
   - **Evidence:** the closed path (`prreview.go:673`) behaves the same way, so this is consistent with it. The `Continue` path requires `gated`.
   - **Severity:** the reviewer asked for exactly this verdict, so I rate it LOW and not a defect to fix.

**Traced and correct**
- **F1, PR-Review `decide`:** on a 404 with a record, `ReadRecord` gives `recordFound` true. `resolveWithParentRule` finds the error is not a token failure and `transientFailure` returns nil, so it returns `Halt` from `githubHaltReason`. `stuckVerdict` takes it, and the record and PR are untouched.
- **F1, Finalize:** `halt(decision.Halt)` adds `LandingStopPrefix`, and no Close call is made.
- **F1, rows:** both F1 rows fail with the defect in place, because Status and Close calls would appear.
- **F1, ordering:** moving `ReadRecord` ahead of the resolve does not disturb the earlier-failure rows. The remaining caller of the changed signature is `decide`, and it is updated.
- **F2:** the clean and dirty rows hang or mismatch without the new case. `stopLiveSession` is safe with an empty `StrandGUID`.
- **F3, cancelled context:** the Origin seam cancels the context, the fake returns `context.Canceled`, and `transientFailure` returns the cancel error. The row fails if that path is removed, since the Warn path would give Done.
- **F4:** the empty-task and local-origin rows assert zero calls. `taskHasCommits` and `checkOrigin` returning early is what keeps them at zero.
- **Gate:** `lym gate test ./internal/landingshed` passed (696 passed, 0 failed).

**Calls made**
1. `grep -A40 "^func Class"` in `internal/shedtransient`: to see whether a bare sentinel counts as transient. My context lacked it.
2. `grep -A30 "^func IsTransient"` in `internal/githubclient`: to confirm the bare sentinel is not transient. My context lacked it.
