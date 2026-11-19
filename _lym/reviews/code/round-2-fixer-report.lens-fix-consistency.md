```yaml
lens: fix-consistency
outcome: reported
```

no findings

The change set agrees with its surroundings. The one sibling I checked, the `recordFindings` doc comment, is now true.

- `internal/gatecli/mutation.go:108` says a failed call records "the error and its partial findings but no changed function or file hash". That now holds for the check's own findings as well as the refresh errors and pre-check not-judged packages.
- The only `diffMutation{` literals are in `mutate.go` and `mutatecheck_test.go:59`. The test one builds a `diffMutation` for `selectWithinCap`. Neither relies on the old empty-outcome return.
- `mutationCall.run` seeds `found.outcome` with `newMutateOutcome()`. A slot error, which returns `d.outcome`, therefore still yields empty lists, as the review's fix key asked.
- No `mutateOutcome{}` literal is left to disagree with the new returns. `findingsSoFar` is used at all three return points.
- `runMutate` on error prints only the error, so its envelope does not change.

Tool calls made:
1. `grep` for `diffMutation{`, `mutateOutcome{}`, "partial findings", "partial counts" and `findingsSoFar`. My context held the edited files but not the other callers, tests or docs.
