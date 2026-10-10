# Card 7 — recover-batch-non-done-exits-non-zero

**Edit:**
- `internal/webstercli#websterCLI.recoverBatchCmd`
- `internal/webstercli/doc.go`
- `contracts/specs/refusal-spec.md`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/claims_test.go`
- `internal/webstercli/verbs_test.go`

**Uses:**
- `internal/webstercli#digestFields`
- `internal/websterengine#RecoveryRetry`

**Intent:** Make `lyx webster recover-batch` exit non-zero whenever the record it returns for is terminal and not `done` (decision D3): `stuck` and `dead` join `failed`.
In the terminal branch of `recoverBatchCmd`, after the fabric sync and after `recovery_retry`, `recoveries` and `warnings` are added to the digest fields, a digest whose status is not `done` goes out through `output.ErrFields` with the same fields (`batch`, `status`, `head_sha`, `deviations`, `dead_reason`, `elapsed_s`, `recoveries`, `recovery_retry`, `warnings`), so `ok: false` and the exit code follow the verb's meaning: the recovery did not recover.
The message names the ladder's way forward: `lyx webster recover-batch NN` once more when `recovery_retry` is true, else end the run stuck naming the batch.
A `done` digest and a running snapshot still exit 0.
Add the paragraph to the verb's `Long` text and to `internal/webstercli/doc.go`.
Add a row to the webster section of `refusal-spec.md`: "recovery not done", triggered when `recover-batch` returns for a terminal record that is `stuck` or `dead` (a `failed` one already carries `batch_failed`), class correctness halt, way forward as above, with the digest fields on the envelope.
In the Master stencil's failure ladder, the sentence "its output is the terminal digest or a refusal" becomes: its output is the terminal digest, exiting non-zero when its `status` is not `done`, or a refusal; Master reads the digest fields, never the exit code, so the rungs are unchanged.
Pin that sentence in the Master stencil's claims in `claims_test.go`.

Tests, in `TestRecoverBatchCmd_RunningThenTerminal` (`verbs_test.go`): the dead-recovery call now expects a non-zero exit with `"ok":false`, `"status":"dead"`, `"recovery_retry":true`, `"recoveries":1` and a message naming `lyx webster recover-batch 1`;
add a terminal `stuck` step, a report with `status: FAILED`, expecting a non-zero exit whose message says to end the run stuck naming batch 1, with the digest fields present;
the done and running calls keep expecting exit 0.
The refusal writes its friction note through `noteRefusals` as every `ok: false` envelope does; no note code changes.

**ImpactSummary:** One CLI verb's exit code on two terminal statuses, with its spec row, stencil sentence and test; an operator script reading the exit code sees non-zero for stuck and dead from now on.
