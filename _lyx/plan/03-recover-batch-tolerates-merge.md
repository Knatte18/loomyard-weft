# Card 3 — recover-batch follows the same head rule, package doc covers both verbs

**Edit:**
- `internal/websterengine#awaitTerminal`
- `internal/websterengine#PersistRecoveryTerminal`
- `internal/websterengine#RecoverResult`
- `internal/websterengine/recoverbatch_test.go`
- `internal/websterengine/doc.go`

**Uses:**
- `internal/websterengine#refuseMidMerge`
- `internal/websterengine#reconcileReportHead`

**Intent:**
Give recover-batch's head cross-check the same D1/D3/D4 rules card 2 gave record-batch, so the two verbs agree.

In `awaitTerminal`, once the digest is terminal (past the `DigestStatusRunning` return) and before the head cross-check, call `refuseMidMerge(deps.Geom.WorktreeRoot)` and return its error, so the batch stays non-terminal while a merge is in progress.
Replace the inline `actualHead != digest.HeadSHA` block with `reconcileReportHead(deps.Geom.WorktreeRoot, digest.HeadSHA, fmt.Sprintf("recovery report for batch %02d-%s", number, slug))`, still only when `digest.HeadSHA` is non-empty;
a non-empty warning goes into the returned `RecoverResult.Warnings` alongside the substrate-cleanup warnings, which the CLI already surfaces.
Update the `RecoverResult` doc comment, which describes `Warnings` as non-fatal substrate-cleanup failures only, to list the moved-HEAD notice too.
`PersistRecoveryTerminal` already feeds `digest.HeadSHA` to `postBatchChecks` and `CardSHAs` (D2); only its comment changes, from "the same value record-batch cross-checks against the worktree's real HEAD" to say both verbs reconcile the report head against HEAD under the merge-only rule and record the batch at the report's head.

In `doc.go`, extend the merge-only head-rule paragraph card 2 added to the fork-return contract section so it covers both bracket verbs that consume a report: recover-batch follows the same acceptance, recording, warning and refusal rules as record-batch.

Tests, in the existing integration-tagged `recoverbatch_test.go` on `newRecoverFixture`, driven through `driveRecoverBatch` as `TestRecoverBatch_SecondCallAttachesAndPersistsDoneDigest` does:
- a recovery report at the worktree's head, then a `--no-ff` parent merge on top before the terminal call: the batch goes terminal, `CardSHAs` holds the report's head, and `Warnings` carries the moved-HEAD notice naming the merge SHA;
- a non-merge commit on top of the report's head: refused naming both SHAs and the merge-only rule, the batch stays non-terminal;
- a conflicting merge left in progress: refused with the `merge --continue`/`--abort` pointer, the batch stays non-terminal.
`TestRecoverBatch_ReportHeadSHAMismatchIsHardError` and the other existing tests pass unchanged.

**ImpactSummary:** recover-batch's terminal path gains the mid-merge refusal and merge-tolerant head check; the equal-HEAD path and substrate cleanup are unchanged.

**Commit:** 3: websterengine: recover-batch follows the merge-only head rule
