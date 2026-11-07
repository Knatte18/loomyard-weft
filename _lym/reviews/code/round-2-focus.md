---
round: 2
focus:
  - "In internal/fabricengine/worktreeseam_integration_test.go, TestWorktreeSeam_Events calls fabricengine.InstallWorktreeSeam at its top level. Settle whether every hubforge.CopyRegistered call the test's subtests reach, through newOriginHub and newMovesPair, runs before that install, so no template's first build runs under the fake seam."
  - "Across the internal/fabricengine integration tests that install a process-global seam or hook, settle whether each takes its registered-template copies before the install, as pattern/PATTERN-test-isolation.md requires."
---

The focus names where to look and what to settle; it states no verdict.
