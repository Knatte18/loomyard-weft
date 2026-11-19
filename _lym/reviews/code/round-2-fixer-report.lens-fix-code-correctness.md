```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1 and F2 fixes (`diffMutation.run`, `mutationCall.run`, `TestMutationStep_RecordFindings`) from the context I already hold.

Findings:

- **LOW**: `internal/gatecli/mutatecheck.go`, `diffMutation.run` and `findingsSoFar`, and `internal/gatecli/mutate.go`, `mutationCall.run`.
  - Summary: no test fails if the F1 change is reverted.
  - Evidence: `TestMutationStep_RecordFindings` feeds `recordFindings` a hand-built `mutationFindings`, so it never goes through `diffMutation.run` or `mutationCall.run`.
  - Reverting to `return mutateOutcome{}, err`, or dropping `found.outcome = outcome` before the `err` check, leaves it green.
  - The review's Rejected section declined an integration row for this, and the fix rule adds a test only for a coverage gap, so this is expected. I report it only because the lens asks whether the tests catch the defect.

Everything else traced correctly:

- **Slot error:** `holdSlot` fails, so `run` returns `d.outcome`. That is the `found.outcome` that `mutationCall.run` seeded with `newMutateOutcome()`, so it has empty lists and no counts.
- **Cancellation after planning:** `run` returns `findingsSoFar()` with the uncovered blocks, the not-judged packages and `runs` already set. `selectWithinCap` has not run, so `NotRun` is 0.
- **`runPackage` failure:** `run` returns `findingsSoFar()`. Survivors and unjudged mutants of packages already judged stay, because `record` appends to `d.outcome` as each package finishes.
- **Concurrency:** `judgeMutants` waits for its workers (`workers.Wait()`) before returning, so `d.runs.Load()` is stable when `findingsSoFar` reads it.
- **`mutationCall.run`:** `found.outcome` is assigned before the error check, and on success the value is unchanged.
- **Callers:**
  - `runMutate` prints only the error on failure, so its envelope is unchanged.
  - `mutationStep.run` reaches `recordFindings` with the partial outcome and the error. That still records no `Functions` or `FileHashes` on error.
- **`TestMutationStep_RecordFindings`:**
  - Dropping the `report.Unjudged` assignment fails every row.
  - Dropping the `append(found.notJudged, found.outcome.notJudged...)` fails every row.
  - Swapping the order of the two lists fails.
  - The expected slices equal the inputs by value, so `cmp.Diff` compares correctly.

Tool calls made beyond context: none.
