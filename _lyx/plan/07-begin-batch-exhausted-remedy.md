# Card 7 — begin-batch-exhausted-remedy

**Edit:**
- `internal/websterengine#existingReportRemedy` -> `func existingReportRemedy(number int, recorded *BatchState, retry bool) (string, bool)`
- `internal/websterengine#BeginBatch`
- `internal/webstercli#websterCLI.beginBatchCmd`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/specs/refusal-spec.md`
- `internal/websterengine/doc.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/webstercli/verbs_test.go`

**Uses:**
- `internal/websterengine#ErrRecoveryExhausted`
- `internal/websterengine#rendersNewAmendment`
- `internal/websterengine#maxRecoveries`
- `internal/websterengine#RecoveryRetry`
- `internal/websterengine#resetVerb`
- `internal/websterengine#freshRestartSteps`
- `internal/websterengine#stepRun`

**Intent:**
D6: begin-batch's existing-report remedy never names a route recover-batch refuses, and follows `RecoveryRetry` for a dead recovery.

`existingReportRemedy` returns the remedy and whether it is the exhausted case, and words every terminal record that is not done in recover-batch's own order, the first matching case winning:
a record of any kind and any terminal status but done, whose `Recoveries` is at `maxRecoveries` and for which `rendersNewAmendment` is false, is the exhausted case, and its remedy says the batch's recoveries are exhausted, so end the run stuck naming the batch, and for the operator `1) <reset --to batch-start --batch NN through resetVerb>; 2) lyx webster recover-batch NN`, or the fresh restart steps `1) lyx webster reset --to start; 2) lyx webster run`;
a dead recovery with `retry` true names `lyx webster recover-batch NN` once more, as today;
any other dead recovery names ending the run stuck naming the batch, never "exhausted";
a stuck or failed record names `lyx webster recover-batch NN`, as today.
The non-terminal and done cases are unchanged.
`BeginBatch` builds the refusal from the pair: the exhausted case wraps `ErrRecoveryExhausted` with `%w` at the front, and the message holds exactly one `way forward:` clause either way.
Only a recovery record is ever terminal dead, so no case keys on a dead fork record.

`websterCLI.beginBatchCmd` maps an error wrapping `ErrRecoveryExhausted` to the envelope flag `recovery_exhausted: true`, exactly as it maps `ErrPlanDrifted` to `plan_drifted`, and its `Long` text gains one sentence naming the flag.
Bound: the flag is set only on the exhausted case and lets Merriam skip nothing; it stops the run where recover-batch would refuse anyway.

Master's begin-batch existing-report rung in `contracts/stencils/webster/webster-template-master.md` gains one sentence: a refusal carrying `{"recovery_exhausted": true}` ends the run stuck naming the batch, as the recover-batch rung does, and its reset steps are the operator's.
The "report already present" row of `contracts/specs/refusal-spec.md` is reworded to the four terminal cases above with the exhausted case first, and the recovery section of `internal/websterengine/doc.go` says begin-batch's exhausted refusal carries `recovery_exhausted`.
PATTERN-no-denied-recovery: the routes named are the ones recover-batch's exhausted refusal already names, and no new command appears.

Rows of `TestBeginBatch_Refusals`, each over a seeded report:
the existing "a report over a terminal dead record ends the run stuck" row, whose record has no recoveries, now expects "end the run stuck naming the batch" and adds "exhausted" to its must-not list;
a dead recovery at the cap with an unrendered amendment expects ending the run stuck naming the batch, never "exhausted";
a stuck fork record at the cap with no unrendered amendment expects `ErrRecoveryExhausted`, "exhausted", ending the run stuck, and `lyx webster reset --to batch-start --batch 01`;
a failed fork record at the cap with an unrendered amendment expects `lyx webster recover-batch 1`, never "exhausted";
a dead recovery at the cap with no unrendered amendment expects `ErrRecoveryExhausted`, "exhausted", ending the run stuck, and the reset-to-batch-start route.
The existing "committed names recover-batch once more" row stays.
`TestBeginBatchCmd_ReportOnDisk` gains a row for a stuck record at the cap expecting exit 1 and `"recovery_exhausted":true`, and its two existing refusal rows assert the flag is absent.

**ImpactSummary:** the remedy function gains a result and its only caller branches on it; the CLI verb gains one flag mapping; one stencil sentence, one spec row and one doc line change.
