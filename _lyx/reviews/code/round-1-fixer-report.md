# Fixer-report, code round 1

## Changes

- **F1** fixed as proposed: the dead arm of `deadEndBatchError` (`internal/websterengine/runlevel.go`) now carries `&& !rendersNewAmendment(bs)`, so a dead recovery holding an unrendered amendment is not refused.
  The row for it is in `TestRun_DeadEndRefusalSparesRecoverableBatches`.
- **F2** fixed with an own change, marked as such.
  The proposed shape was kept in outline: `RecoveryRetry` now wraps `recoveryRetryRead`, which returns a failed git read as an error, and `deadEndBatchError` returns it as a plain error, never `ErrBatchDeadEnd`.
  The reviewer did not propose the way-forward text; the first version said "transient, re-run", which the self-check showed wrong for a start commit that is permanently gone.
  The final text names the re-run and, if the read keeps failing, the batch-start reset.
  `RecoveryRetry`'s behavior and its other caller are unchanged.
- **F3** fixed as proposed, with one deviation: the negative rows are a separate table-driven test, `TestRun_DeadEndRefusalSparesRecoverableBatches`, and not rows of `TestRun_RefusesBeforeSpawn`, because the latter's harness requires an error from every row and these rows reach Master.
  Rows: a dead recovery with no counted recovery (retry true), a stuck batch at the cap with an unrendered amendment, a dead recovery at the cap with one, and a done batch.
  A row for the F2 git-failure branch went into `TestRun_RefusesBeforeSpawn`.
- **F4** fixed as proposed: the doc comment of `TestRecordBatch_MalformedReportYAMLErrors` is back above its function.
- **F5** fixed as proposed: `verifyEveryBatchDone` reports a waiting batch as `NN-slug (waits on its parent: <need>)`.
- **F6** fixed as proposed: `head_sha` starts its own line in `webster-body-implementer.md`.
- Self-check follow-ups: `doc.go` states the revised dead-end rule and the git-fault refusal; line 39 of `webster-body-implementer.md` also got a sentence break.

Commits: F4, F6 and the F1/F2/F3/F5 changes (one commit, as they share files) plus one follow-up commit for the self-check changes.
`lyx gate test ./internal/websterengine` passes.

## Self-check

- fix-consistency, LOW: `doc.go` states the old dead-end rule. Fixed as proposed.
- fix-code-correctness, LOW: the git-failure branch has no test. Fixed as proposed (row in `TestRun_RefusesBeforeSpawn`).
- fix-proposal, MEDIUM: "transient" mislabels a permanent git failure and hides the reset route. Fixed with an own change (the way forward names both the re-run and the batch-start reset, and the branch is not limited to a failed `HeadSHA`, which the dead rule still needs).
- fix-proposal, LOW: no test for the git-failure branch. Duplicate of the correctness finding, fixed with it.
- fix-proposal, LOW: the F1 guard is broader than "at the cap". The fork judged it correct; no change, as the guard matches the spawn rule below the cap too.
- fix-affected-set, LOW: line 39 of the implementer stencil holds two sentences, outside the diff. Fixed as proposed, since the change is trivial and harmless.
- fix-proposal's pre-existing note (a dead fork-kind batch is refused as no retry): not a finding of this change set; left as it is.

All four forks returned reports.

## Disputed

None.

## Deferred

None.
