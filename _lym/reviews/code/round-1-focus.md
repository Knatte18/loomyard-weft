---
round: 1
focus:
  - "Each fold commit (`git log` over the range): does its message name the assertions the folded row alone made, and does each named assertion still stand in the remaining test in `internal/fabricengine`?"
  - "`livestate_matrix_test.go` `runCell` and `livestate_verbs_selftest_test.go`: do the three preconditions (Add's launcher blocker is a directory, Checkout's branch differs, Pull's bare advanced) run right after the verb's arrange and before the state is applied, and is any `TestVerbCases_CleanState` check lost?"
  - "`boltpullfirst_integration_test.go` (`TestBolt_PullThenCommitWritten`): in the two-row tables and the later fix commits, is every assertion of each former subtest still made on the row that used to make it, including the diverged-conflict log-count and logger capture?"
  - "`pushclasses_integration_test.go` (`TestPushFailureClasses`): do settle and backoff rows assert on the first push, and does the backoff table keep each former row's expectations?"
  - "`anchorchange_integration_test.go`, `housekeeping_integration_test.go`, `originreconcilemoves_integration_test.go`: with a shared `hubforge.CopyHub`, a shared pair or a merged probe, can state left by one row change the outcome of another?"
  - "Does each test keep one `tiergate.Require` at the lowest fitting tier after the fold (`PATTERN-test-speed`)?"
---

Scope is the D6 folds only; the diff touches test files in `internal/fabricengine`.
