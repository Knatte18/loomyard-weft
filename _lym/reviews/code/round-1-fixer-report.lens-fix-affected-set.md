```yaml
lens: fix-affected-set
outcome: reported
```

Task: run the fix-affected-set lens, checking that each set-premise fix covers every member.

Findings: no findings.

Findings searched: F1 (callers of outputAboveHeader: one call site, passes failure.Package; other ParseVerifyFailures consumers read Tail only), F2 (newCommand callers: Command() and the test helper, both updated; RunGate builds its testRequest directly), F5 (retryOnShortage: one production caller, one test). F3 and F4 not checked, their premise is not a set.

Tool calls (2 of 6): grep for the three function names over Go files; grep for FailureKindTest, ParseVerifyFailures( and .Tail.
