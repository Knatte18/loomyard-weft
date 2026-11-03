# Fixer report — code round 1

## Changes

- **F1** (fixed as proposed), commit `websterengine: fix F1`.
  - `runwrites_test.go`: the "written worktree paths" subtest gains a split-root call with `PaneCwd` = `<worktree>/backend`, asserting `backend/a.go` and `top.go` and a dropped outside write.
  - `suspect_test.go`: `TestCheckSuspectPaths` gains a subtest running `trackedRel` and `suspectBlobs` with a subdirectory pane cwd, including a `../tracked.txt` path that leaves the pane cwd but stays in the worktree.
  - `batchfail_integration_test.go`: `TestFailBatch_RecordsSuspectBlobs` gains a `failBatch` call with `PaneCwd` = `<repo>/internal` and the relative path `x.go`.
  - No new test functions.
- **F2** (fixed as proposed, with one addition), commits `webstercli: fix F2` (two).
  - `newVerbsFixture` seeds the plan with `Root` set to the anchor and the committed `backend/README.md` as an Edit target, so a dispatch-time lookup against the anchor reports `path-missing`.
    The addition to the proposal came from the self-check: a Create-only plan is skipped by the `path-missing` check, so it would not have pinned `ValidateDispatch`.
  - `TestRecordBatchCmd_Envelope` and every other test committing the fixture's Create target now commit `backend/internal/only/new.go`.
  - `TestRebaselineCmd_EditedCardOfFailedBatchThenRecover` keeps `Root` in its two plan edits, so only the card changes.
  - The fixture's doc comment names the root.

Gate: `lym gate test --tier 1 ./internal/websterengine ./internal/webstercli` passed (464 targets, 0 failed) before the last comment-only edit.

## Self-check

- fix-code-correctness, MEDIUM, begin-batch dispatch not pinned by a Create-only plan: fixed (Edit target added; also raised by fix-proposal, merged).
- fix-proposal, MEDIUM, same finding: merged into the line above.
- fix-consistency, LOW, root-less Create commits at the fingerprint, failed-batch, archived-report and fabric-sync tests: fixed (all now commit `backend/internal/only/new.go`).
- fix-consistency, LOW, `newVerbsFixture` doc comment silent on the root: fixed.
- fix-consistency, LOW, plan rewrites that drop `root:` (conflict, edited-card and drift rows): disputed, premise false; see below.
- fix-affected-set: no findings.

## Disputed

- fix-consistency LOW (plan rewrites without `root:`), ground `premise false`: the rows that rewrite the plan expect a fingerprint or `plan_conflict` refusal, or read no overview.
  The gate passes with them unchanged, and the one row that did depend on the overview (`TestRebaselineCmd_EditedCardOfFailedBatchThenRecover`) was fixed.

## Deferred

None.
