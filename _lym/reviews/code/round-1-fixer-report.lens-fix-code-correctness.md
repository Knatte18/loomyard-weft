```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness check of this round's change set.

One finding.

**MEDIUM: `internal/gatecli/counters.go`, `observeSummary` (F11) and its new doc comment**
The comment says an open summary also ends "at an invocation boundary", but only `results: trx` tiers have such a boundary in the counter.
- **Evidence:** `runInvocations` in `tierrun.go` calls `vstest.startInvocation()` only inside `if run.tier.Results == gateslot.ResultsTRX`, and only when the temp directory was made. `replaceInvocation` likewise runs only when there is a results directory.
- **Effect:** on a `vstest` tier without `results: trx`, a summary that gets no `Total time:` line (a killed or truncated project output) now stays open into later invocations. Any later bare `Passed: N`, `Failed: N` or `Skipped: N` line is then added to the counts, until a one-line summary or a `Total time:` line arrives. Before this change, the first foreign line closed it.
- **Possible fix:** call `startInvocation` for every `vstest` invocation, whether or not it has a results file. Its other effects are marking the counts and failures that `replaceInvocation` would replace, and clearing `inFailure`.

I found nothing wrong in the other changes (traced by reading):
- **Timeout bound:** all three `withTimeoutMin` callers are updated. The new wall-clock row catches a mutant that drops the wall term, though it does so by hanging on `<-ctx.Done()` until the test times out, as the old rows would have.
- **Trailing `--filter`:** the refusal and its exact-text row agree.
- **New tests:** the `otherTiers` rows, the budget-plus-caller-filter row and both counter rows each fail with their defect in place.
- **Key names and notice text:** the helpers render the same strings the tests expect, and the filtered notice's new argument order is correct.

I made no tool calls; my context already held everything the lens needed.
