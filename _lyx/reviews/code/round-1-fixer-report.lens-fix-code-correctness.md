```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fix, two new `TestRecoveryRetry` rows for F1.

Run of the new rows against `RecoveryRetry` (`recoverbatch.go:416-440`):

- **Fork row.** The input is `deadRecovery(0, "")` with `Kind = "fork"`, stored at `Batches[1]`.
  - `bs` is non-nil and `recoveries` is 0.
  - `bs.Kind != "recovery"` is true, so it returns `(false, 0)`.
  - The row wants `wantRetry` false (the zero value), `wantRecoveries` 0 and `wantWarn` false, so it matches.
- **Non-terminal row.** The input is the same with `Terminal = false`.
  - `!bs.Terminal` is true, so it returns `(false, 0)`, which matches the row.
- **Defect in place.** Suppose the `recoveries == 0` branch moved above the kind and terminal checks.
  - Both rows would return `(true, 0)`, so each fails on `wantRetry`.
  - Each row therefore pins the position the review named.
- **Failed calls.** Neither row reaches `committedSince`, so no git read happens and no Warn line is logged. `wantWarn` false holds.
- **Callers.** The fix changed only test code, so no signature or behaviour has a caller to break.
  - The rows use `deadRecovery`, `fakeGit` and the table fields as the existing rows do. The unused `g` parameter matches the existing `deadRecovery(0, "")` row.
- **Gate.** `lyx gate test ./internal/websterengine` passed 1025 of 1025 in the parent's run.

No findings.

Further tool calls: none. My context already held the test file and `RecoveryRetry`.
