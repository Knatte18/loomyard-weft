# Card 8 — record-batch-audit-dispositions

**Edit:**
- `internal/websterengine/recordbatch.go`
- `internal/websterengine/recordbatch_test.go`

**Uses:**
- `internal/websterengine/audit.go`
- `internal/websterengine/auditledger.go`
- `internal/websterengine/cardverify.go`
- `internal/websterengine/batchfail.go`

**Intent:**
Replace record-batch's "any audit violation refuses" with D5, D6 and D8's dispositions, so a policy finding warns once and a correctness finding fails the batch with `recover-batch` as its way forward, never a refusal that repeats on every retry.

`RecordDeps` gains `VerifyTimeout time.Duration` (zero means `DefaultCardVerifyTimeout`).
`RecordResult` gains `Failed bool`, set alongside a failed `Digest` when the batch was failed.

New order inside `RecordBatch`, after the bracket checks, settle retry and `ClassifyAttribution`, which stay as they are:

1. Collect `CheckParent` violations (identity from `bs.SessionID`) and `CheckFork` violations for each new transcript, and drop every one whose `findingIdentity` is already dispositioned (D8: a parent finding is consumed by the first record-batch that reports it).
2. Classify each remaining finding with `ClassifyViolation` against `deps.Geom`.
   A probe error returns before any mutation (D2's grouped transient I/O row).
3. Any correctness finding: every remaining finding is marked `recordFailedFinding`, and `failBatch` takes the batch terminal-failed with one reason per finding (`Error()` text) and every correctness `Path` as a suspect path, consuming the new transcripts.
   Return `RecordResult{Digest: <failed digest>, Failed: true, Warnings}` together with the `*BatchFailedError`.
   This runs whatever the report says, and with no report at all.
4. Otherwise advance attribution as today.
5. No report: record each policy finding as a batch warning (`recordBatchWarning`) and return `NoReport` with those warnings.
6. Report present: parse it, then run the mid-merge, dirty-worktree and head reconciliation steps unchanged.
7. Policy findings on an `OK` report: run `rerunCardVerifies` over the batch's cards on `deps.Geom.WorktreeRoot`.
   Any failure makes every policy finding correctness class for this batch (D5): `failBatch` with the findings and the verify failures as reasons, as in step 3.
   All pass: record each as a batch warning and carry its text on `Warnings`.
   A `FAILED` report records the policy findings as warnings with no verify re-run.
8. Continue to `postBatchChecks` and the terminal digest as today.

Update the file's header comment and the doc comments of `RecordDeps`, `RecordResult` and `RecordBatch` to describe the dispositions.

Tests in `recordbatch_test.go` (integration), each from a real scratch repo with state as the verbs write it:

- `fabric-reference` in the fork's transcript, OK report, a card whose `**Verify:**` passes → batch terminal `done`, `Warnings` carries one `audit warning (fabric-reference)` line, `bs.AuditWarnings` has one entry; a second `RecordBatch` call on the now-terminal batch adds no second warning.
- The same finding with a card verify that exits 1 → batch terminal `failed`, the report file moved to its archive name, `errors.Is(err, ErrBatchFailed)`, message names `lyx webster recover-batch`.
- A `named-spawn` parent finding at batch 2 → batch 2 done with one warning; batch 3 (same session, same whole-session audit plus its own fork) records done with no refusal and no repeated warning.
- A Master `parent-write` to `_lyx/webster/state.json` → correctness, batch failed, the path in the digest's reasons.
- A `fork-contract-write` → batch failed, report archived.
- A `parent-write` to a path outside the worktree → policy warning, batch done.
- `TestRecordBatch_Regression20260930_ForkAuditFalsePositive`: the 2026-09-30 incident's fabric-reference finding on an otherwise clean batch now records done with a warning instead of refusing every retry.

Rewrite the existing tests that asserted a hard error for a violation (for example `TestRecordBatch_ParentWriteOutsideContractFilesErrors`) to assert the new disposition.

**ImpactSummary:** Changes record-batch's audit outcome for every violation class; its only callers are webstercli's record-batch verb (card 12) and the engine tests updated here.
