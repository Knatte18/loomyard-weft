```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Per change, the set found and whether it is covered:**

1. **`RecoveryRetry` verdict, zero-count early return** (`internal/websterengine/recoverbatch.go:416-440`).
   - **Production readers, all covered:** `internal/webstercli/status.go:75`, `internal/webstercli/recoverbatch.go:344` (which feeds `recoveryNotDoneMessage` at `:348`) and `internal/websterengine/beginbatch.go:556`. All three read the function alone and none carries its own copy of the old rule.
   - **Fixtures at zero counted recoveries, covered:**
     - Updated to `Recoveries: 1`: `beginbatch_test.go:187`, the old no-retry row.
     - Added: the `beginbatch_test.go:198` row, the `recoverbatch_test.go` `TestRecoveryRetry` row and the `cli_test.go:846` status row.
     - Untouched and unaffected: `recoverbatch_test.go:56, 175, 1019, 1207, 1220` and `beginbatch_test.go:645, 666`. They are recover-batch or begin-batch spawn and attach rows. None asserts `recovery_retry` or the begin-batch remedy text.
     - `verbs_test.go:1123` asserts `"recoveries":1` and `:1169` asserts `"recoveries":2`. Neither is touched by the zero-count branch.
   - **Edge case, not a finding:** the "record with no start" row now passes `Recoveries` 1 or 2. `deadRecovery(n, start)` (`recoverbatch_test.go:1699`) takes the count from its caller, so the row still reaches the empty-start check.

2. **`existingReportRemedy` retry wording** (`beginbatch.go:424`).
   - Two tests assert on the begin-batch remedy text: the changed no-retry row at `beginbatch_test.go:190` and the new row at `:198-205`. The old string "since the dead recovery committed work of its own" now appears nowhere in the tree, per grep.
   - No doc, stencil or help text quotes the old string.
   - Covered.

3. **Master stencil dead rung** (`contracts/stencils/webster/webster-template-master.md:125-126`).
   - The other `recovery_retry` mentions are consistent with the new rule. They are `:63-65` (the status read), `:127` (the false rung) and `:156` (the begin-batch dead-batch line).
   - Dropping "that second recovery is the last" is correct, since a reset batch can now take a first retry. The cap is still stated through the other rungs and through `recovery_exhausted` at `:157`.
   - Nothing else is weakened. Covered.

4. **Docs and comments:**
   - Updated: `internal/websterengine/doc.go:432-439`, the `recover-batch` long help (`internal/webstercli/recoverbatch.go:99-105`), the `RecoveryRetry` doc comment, the `RecoveryStartSHA` comment at `state.go:330` and the `existingReportRemedy` comment.
   - Generic and still accurate: `internal/webstercli/doc.go:50-51` ("once more when recovery_retry is true").
   - Outside scope, the one remaining hit: `docs/research/discussion-bench.md:112` mentions `recovery_retry: true` only for exit codes.
   - Covered.

**Findings:** none.

**Tool calls (2):**
- A repo-wide grep for the old wording, `recovery_retry`, `RecoveryRetry` and `recoveryNotDoneMessage`, to find every reader, assert and doc outside the touched lines.
- A grep for dead-status test fixtures, plus `sed -n 150,158p` of the stencil, because my context lacked the other dead-batch fixtures and the stencil's begin-batch rung.
