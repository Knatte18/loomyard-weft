---
round: 2
focus:
  - "internal/landingshed/finalize.go, judgeFixerChange and letThrough: check that every route out of a fixer turn (HEAD equal to Baseline, within the caps, fix review approved, operator accepted) leaves the attempt record's Judged consistent with the task head, and that a later parent merge moves Baseline accordingly."
  - "internal/landingshed/finalize.go, rebaseAfterParentMerge: check that Baseline moves only for a merge that brings parent commits onto a TaskHead equal to Judged, and stays put for a no-op merge or a differing TaskHead, and that the lock-freed tests in finalize_gate_test.go pin each branch."
  - "internal/landingshed/finalize.go, runFixReview and freeLock: check that the parent lock record is absent while a fix review session runs, and that a test observes this."
---

Focus targets the code a round-1 fix is likely to touch.
