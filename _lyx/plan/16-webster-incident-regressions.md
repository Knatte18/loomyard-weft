# Card 16 — webster-incident-regressions

**Edit:**
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/parentmerge_test.go`

**Intent:**
Pin the 2026-09-30 wedges so they cannot return, beside the fork-audit false-positive regression card 8 adds.

- `TestRun_Regression20260930_BegunUnrecordedBatchResumes` (runlevel_test.go): state records batch 1 begun, non-terminal, with its Create target already committed; `Run` passes entry validation and reaches the Master spawn with no `create-already-exists` refusal.
- `TestBeginBatch_Regression20260930_ReBeginOfBegunUnrecordedBatch` (beginbatch_test.go): the same state, `BeginBatch(1)` succeeds and keeps the recorded StartSHA.
- The parent-merge ring is already pinned by `TestParentMergeBetweenForkCommitAndRecordBatch`; add a doc-comment line naming it the 2026-09-30 parent-merge-ring regression, and extend it so the batch after the merge-in is recorded, proving no verb in the ring refuses.

Each test is written against the behaviour on this branch; a test that already passes before this card is still added, since its job is to keep that behaviour.

**ImpactSummary:** Test-only; three integration test files in websterengine gain or extend a regression test.
