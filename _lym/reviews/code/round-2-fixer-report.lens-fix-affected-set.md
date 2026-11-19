```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the F1 and F2 fixes cover every member of their sets.

No findings.

- **F1 (return sites of `diffMutation.run`):** the grep finds no `mutateOutcome{}` left in `internal/gatecli`. All three returns in `run` after the slot hold now go through `findingsSoFar`, at `mutatecheck.go:187`, `:202` and `:205`. The slot error returns `d.outcome`, which `mutationCall.run` seeds with `newMutateOutcome()`.
- **F1 (callers):** `check.run()` has one caller, `mutate.go:298`, which now assigns `found.outcome` before the error check.
- **F1 (error-path reader):** the error path in `runMutate` prints only the error. `mutation.go:103` passes `found` to `recordFindings` on both the error and success paths.
- **F2 (assignments the test must pin):** `recordFindings` assigns Mutants, Survivors, Unjudged, Uncovered and NotJudged. The fixed test now asserts Unjudged and NotJudged in every row, as the review asked. Survivors and Uncovered stay empty in every row, which the review did not ask for. Pinning them would be extra, so I did not raise it.

Further tool calls:
1. `grep` for `mutateOutcome{}`, `.run()` and `findingsSoFar` in `internal/gatecli`, to find every return site and caller of the changed `run`.
2. `grep` for `recordFindings` and `found.outcome` in `internal/gatecli`, to find the other readers of the outcome the fix changed.
