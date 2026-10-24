```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal (two zero-count rows, one `Kind` "fork" and one non-terminal), the only finding fixed as proposed.

**Findings**

1. LOW. The doc comment above `TestRecoveryRetry` (`recoverbatch_test.go:1566-1568`) no longer lists every false case the table pins.
   - It names the false outcomes: no commits of its own, a fork's earlier commits, a record with no start, a HEAD a reset moved off the start, merge-ins only, the cap, and a stuck or failed recovery.
   - The two new rows, a non-recovery record and a non-terminal record that a batch-start reset cleared, fit none of those phrases.
   - "A fork's earlier commits" is about commits, not a fork-kind record, so a reader could take the new "fork record" row for a duplicate of it.
   - The fix is to add "a non-recovery or non-terminal record" to the false list.

**Challenge of the proposal itself**

- Both rows fail if the `recoveries == 0` branch moves above the kind and terminal checks. A fork-kind or non-terminal record at zero recoveries would then return `(true, 0)`, against the wanted `(false, 0)`. They pin the branch's position.
- The expected `recoveries` of 0 matches the code, because `recoveries = bs.Recoveries` is set before the checks.
- Neither row touches git, so `wantWarn` false and the `fakeGit` state are irrelevant to them.
- `deadRecovery` returns a fresh pointer each call, so mutating it leaks nothing into other rows.
- No breakage elsewhere. The only other effect is a doc-comment staleness, covered in finding 1.

**Tool calls:** none. My context already held the test file and `RecoveryRetry`.
