# Card 2 — record-batch tolerates a parent merge after the fork's commit

**Edit:**
- `internal/websterengine#RecordBatch`
- `internal/websterengine#RecordResult`
- `internal/websterengine#postBatchChecks`
- `internal/websterengine/recordbatch_test.go`

**Uses:**
- `plan:internal/websterengine#refuseMidMerge`
- `plan:internal/websterengine#reconcileReportHead`

**Intent:**
Stop a parent merge-in that lands between a fork's commit and its `record-batch` call from wedging the run (decisions D1–D4).

In `RecordBatch`, right after the report parses and before the dirty-worktree check, call `refuseMidMerge(deps.Geom.WorktreeRoot)` and return its error unchanged, so a batch stays non-terminal and retryable while a merge is in progress (D4).
Replace the inline `actualHead != report.HeadSHA` cross-check with `reconcileReportHead(deps.Geom.WorktreeRoot, report.HeadSHA, "batch report "+reportPath)`; a non-empty warning is appended to `warnings` (D3).
From there on the batch's head is `report.HeadSHA`, not the post-merge HEAD: pass it as `postBatchInputs.HeadSHA` (so the delta is `StartSHA..report.HeadSHA` and drift's triggering SHA is the fork's own commit) and record `bs.CardSHAs = []string{report.HeadSHA}` (D2).
`postBatchChecks` itself is unchanged: its done-checks and drift tree resolution still read the worktree as it stands, the merged tree.
When HEAD equals the report's `head_sha` the call's behaviour and warnings are byte-for-byte what they were.
Update the `RecordResult.Warnings` doc comment to list the moved-HEAD notice among the non-fatal warnings, and the `postBatchChecks` comment that says HeadSHA is "cross-checked against the worktree's real HEAD" to say it is the reconciled report head.

Tests, in the existing integration-tagged `recordbatch_test.go` on `newRecordFixture`, with a helper that builds a parent-side commit off `fx.StartSHA` on a side branch and merges it `--no-ff` into the fixture's branch after the fork's commit:
- one parent merge after the fork's commit: the batch goes terminal, `CardSHAs` is `[fx.HeadSHA]`, and `Warnings` holds exactly one entry naming `fx.HeadSHA`, the merge commit's SHA and the new HEAD;
- two successive parent merges: terminal, and the one warning names both merge SHAs;
- the parent side adds a Go file with a symbol outside the card's targets (set the card's `Targets` as `TestRecordBatch_ScopeGuardFindingsLandInWarnings` does): no `scope-outside-plan` finding and no drift or bind finding names it — the only warning is the moved-HEAD one;
- a non-merge commit after the fork's commit, alone and after a merge: refused, the error names both SHAs and the merge-only rule, and the batch stays non-terminal with `CurrentBatch` unchanged;
- a fast-forward onto non-merge commits: refused the same way;
- a conflicting parent merge left in progress: refused with the `merge --continue`/`--abort` pointer and the batch stays non-terminal; after the conflict is resolved and committed, the same `RecordBatch` call succeeds and records `fx.HeadSHA`.
A refused call's state is never persisted by the CLI, so the retry runs against the state as it stood before the refused call (restore `SeenForkTranscripts` and the batch's `ForkTranscripts`), with the fake engine scripted to report the same fork transcript again.
Existing tests, `TestRecordBatch_HeadSHAMismatchErrors` included, pass unchanged.

**ImpactSummary:** changes only what `RecordBatch` accepts and records when HEAD moved; the equal-HEAD path is unchanged and `postBatchChecks` changes only in a comment.

**Commit:** 2: websterengine: record-batch tolerates a parent merge after the fork's commit
