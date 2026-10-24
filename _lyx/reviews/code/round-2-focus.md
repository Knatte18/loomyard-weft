---
round: 2
focus:
  - "In internal/gateslot/runlog.go, `FailedTier` and its consumers (internal/landingshed/verifygate.go, internal/landingshed/doc.go, internal/verifytree/doc.go, pattern/PATTERN-verified-tree.md): when several runs fail, a call exits with the first failing run's code; settle whether the tier `FailedTier` names and every doc sentence about it agree with that exit code."
  - "In internal/gatecli/run.go, `runTest`'s loop on `end.halt`: settle whether any halt, including a cancelled context where the run itself exited 0, can return 0 with later runs unrun, and whether a halt after earlier failures keeps the first failure's code and prints the closing line naming every failed run."
  - "In internal/gatecli/test_integration_test.go, the guard-after-failing-tier row: settle whether a failing guard run, its `guard tierN` label in the closing line and the first-failure exit rule across a listed failure and a guard failure are pinned."
---
