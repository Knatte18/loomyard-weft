```yaml
lens: fix-affected-set
outcome: reported
```

Task: lens fix-affected-set. I reran the set-premise searches for F6 (stale "latest failure" wording and callers of the notice renderer) and F1 (sites that set `gateEndCause`).

Findings: no findings.

- **F6:** `verifyBudgetNotice` has one caller, `noticeIfSpent`, and it now passes `failures`. No stale "latest failure" wording is left in the notice, in the `verifygate.go` or `doc.go` comments, or in `template.yaml`. The remaining "latest" hits are unrelated: `failedBefore` (the darn form's latest failed run), two test messages, and webster's own report text.
- **F1:** three sites set `gateEndCause`: wait expiry (`wait.go:408`), no input box (`wait.go:611`) and re-prompt send failure (`wait.go:630`). The fix's tests cover all three, with the generic session-end text asserted in `TestGate_DeadlineExpiresBetweenAttempts` and the empty cases in `TestGate_EndedEarlyStaysEmpty`.
- **Unchecked:** F2, F3 and F5 premises are not sets, and F4 is a single comment move.

Tool calls:
1. grep for stale "latest failure" wording and callers of `verifyBudgetNotice`: my context lacked a repo-wide search for the old wording.
2. grep for the `gateEndCause` assignments: my context lacked a complete list of the sites that set it.
