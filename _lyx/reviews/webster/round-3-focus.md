---
round: 3
focus:
  - "In internal/loomcli/smoke_test.go, the subtest \"origin record self-heals after a crash between write and commit\" edits the records worktree's index with git rm and git commit; settle whether any driver started by an earlier subtest of TestSmokeBootstrapLifecycle can still be alive and writing to that worktree when those git calls run, and whether the subtest stops every such driver first, as the \"died driver proceeds\" subtest does through killDriversAndClearLog."
---

The focus names the one place the publish verify failure points at.
