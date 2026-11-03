---
round: 2
focus:
  - "internal/websterengine: trackedRel and suspectBlobs (suspect.go), writtenWorktreePaths (runwrites.go) and failBatchInput (batchfail.go) take PaneCwd and WorktreeRoot. Settle whether a test runs each with PaneCwd as a subdirectory of WorktreeRoot, so that swapping or misusing the two roots fails it."
  - "internal/webstercli/verbs_test.go newVerbsFixture runs in a non-dot anchor hub. Settle whether the plan it seeds sets root to the anchor directory with card targets existing only under it, so that resolving BeginBatch's dispatch check (beginbatch.go) or RecordBatch's done checks, drift and DiskSizes (recordbatch.go) against the anchor fails a test."
---

Both entries target test coverage of the anchor-versus-worktree-root split.
