```yaml
lens: fix-consistency
outcome: reported
```

Task: checked the fix's change set for disagreement with its surroundings.

1. **MEDIUM**: `internal/gatecli/tierrun.go` `filteredNoTestNotice` (F12). The package doc still describes the old text. `internal/gatecli/doc.go:353` says "When the words after `--` carry a test filter, the line says instead that no test of the tier matched it." The line now says no test of the project matched. The change was made in the code and its test, but not in this doc.

2. **MEDIUM**: `internal/gatecli/counters.go` `observeSummary` doc comment (F11), "The summary ends at its `Total time:` line, at a one-line summary or at an invocation boundary."
   - The only boundary the counter sees is `startInvocation` or `replaceInvocation`.
   - `runInvocations` calls `startInvocation` only inside the `run.tier.Results == gateslot.ResultsTRX` branch (`tierrun.go:555`).
   - So for a `vstest` tier without `results: trx`, a summary that never prints `Total time:` (an aborted or killed host) stays open into the next invocation. Before the fix, the first foreign line closed it.
   - A later bare `Passed: N` line would then be counted, and the comment's bound does not hold for those tiers.

I found no other disagreement in what I searched:
- `build.go`, `copies.go` and `tierrun.go` all use `withTimeoutMin` for `timeout_min`. The remaining `context.WithTimeout` calls bound other things.
- The refusal lists in `planTierRuns` and `doc.go` agree with the code.

Tool calls (1 of 6):
- One grep across `internal`, `docs`, `contracts` and `pattern` for `context.WithTimeout`, the summary and notice wording, and `--filter`. My context did not show these other sites.
