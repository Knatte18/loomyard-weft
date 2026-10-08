# Card 7 — begin-batch-one-remedy

**Edit:**
- `internal/websterengine#BeginBatch`
- `internal/websterengine/beginbatch_test.go`
- `internal/webstercli/verbs_test.go`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/claims_test.go`
- `contracts/specs/refusal-spec.md`

**Intent:** begin-batch's existing-report refusal names the one remedy the recorded state calls for, instead of listing several for Master to choose from (D10).
The refusal keeps naming the report and the record it saw, and its way forward is picked from the record:
- begun, not terminal, `Kind` fork: `lyx webster record-batch NN`, after fixing whatever its last refusal named;
- begun, not terminal, `Kind` recovery: `lyx webster recover-batch NN`;
- terminal `stuck` or `failed`: `lyx webster recover-batch NN`;
- terminal `dead`: the batch's recovery is exhausted, so end the run `stuck` naming the batch, as Master's dead rung does;
- terminal `done`: the batch is finished, so begin the next batch.
The branch that archives a report with no begin record is unchanged.
Master's template rewrites the `begin-batch` already-has-a-report rung to follow the one remedy the refusal names, dropping its own record-batch-then-recover-batch fallback text.
`TestBeginBatch_Refusals` in `beginbatch_test.go`, the existing table of begin-batch's entry refusals, carries the five recorded states as rows: its terminal and begun non-terminal report rows are reworded, and rows are added for the recovery-kind begun record, terminal `stuck`/`failed`, terminal `dead` and terminal `done`, each asserting its remedy exactly and that no other verb is named; `TestBeginBatchCmd_ReportOnDisk` in `verbs_test.go` is updated for the reworded message.
`claims_test.go` pins the rewritten rung, and `refusal-spec.md`'s "report already present" row lists the five remedies.

**ImpactSummary:** Rewords one begin-batch refusal and its Master rung; the begin-batch decision itself is unchanged.
