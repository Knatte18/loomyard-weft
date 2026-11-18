```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F1:** `recordsSideNotLanded` has a single caller, at `pushanchored.go:233`, and the fix updates it. `errRecordsUnpushedPastPush` is read only in `pushanchored.go`. Every doc line on the records re-push (`pushanchored.go:112` and `doc.go:1135`) now sits beside the added owed-push lines.
- **F3:** No other text lists the old "before a missing branch or unreachable origin" wording of the `BackoffSleep` doc.
- **F2:** I did not search for other tests that pass a no-op `BackoffSleep` into the records re-push, so that part is unchecked.

Tool calls:
1. One `grep` over `*.go` and `*.md` for the symbol and phrase sets above. My context lacked a repo-wide view of the callers and the doc phrases.
