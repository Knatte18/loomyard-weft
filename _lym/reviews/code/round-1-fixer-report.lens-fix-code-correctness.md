```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed `wantErr` row and doc comment in `gotohook_integration_test.go` (lens fix-code-correctness).

No findings.

- **Wording matches the code.** `prReviewClearFailure` formats `could not %s: %w`. With `step` = `stop the recorded PR-Review session`, the error contains `could not stop the recorded PR-Review session: shuttle: stop strand guid-1: probe reed before removing it`. The `%w` cause is the error from `StopStrand`, whose `FindRun` and `removeStrandIfPresent` paths end in the `probe reed before removing it` wording from `stop.go:229`.
- **The row fails with the defect in place.** A `stepStopRunner` failure would read `…session: load the shuttle and reed config to stop the session: …`, which no longer contains the new substring, so the row catches a runner that was never built. The gate also passed at tier 1 with the new row.
- **Callers are unaffected.** The change is test-only, and the row name and doc comment now agree. The record-kept assertion (`withStrand`) still holds, since `clearPRReviewOnGoto` returns before `ClearReviewSession` on a stop error.

Tool calls: one `Read` of `internal/loomcli/prreviewgoto.go`, because my context lacked the hook's stop closure and the `prReviewClearFailure` format string.
