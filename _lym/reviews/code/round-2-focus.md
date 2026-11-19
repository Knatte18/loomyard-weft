---
round: 2
focus:
  - "internal/landingshed/finalize.go, `landedDiffEmpty` and `recordLanding`: in a one-repository hub, does a landing that carries no code still read as empty when the parent records commit (`commitParentRecords`, Step 2b) lands after `parentHeadMerged` is read? Check that records paths, as fabric's `RecordsPathsAt` names them, never count as a changed path, and that no `_lym` literal is hard-coded."
  - "internal/landingshed/finalize.go, `recordLanding` on a Finalize re-step after a failed parent push: when the parent head already holds the task's work, is the previously recorded `EmptyDiff` flag kept rather than recomputed from an empty `ChangedPaths(L, L)`?"
  - "internal/landingshed/finalize_test.go, `TestFinalize_RecordsWhetherTheLandedDiffIsEmpty`: does the test and each subtest follow `PATTERN-test-speed` on `t.Parallel`, and do the rows pin the records-only and re-step cases the code now handles?"
  - "internal/darnengine/config.go `LoadConfig` and internal/darnengine/spec.go `reportPassage`: is a `report_dir` under a records tree refused with the way forward named, and does a test reach the passage's read and fill error path?"
---
