---
round: 3
ledger:
  - key: orch-dry-run-creates-state-lock
    rounds: [1]
    status: resolved
  - key: cleanupreport-applied-comment-misdescribes-field
    rounds: [1]
    status: resolved
  - key: fabricvocab-missing-tmuxkit-testmain
    rounds: [2]
    status: resolved
---

Round 1 raised the dry-run state-lock and `CleanupReport.Applied` comment findings, both resolved by round 2.
Round 2 raised the missing `TestMain` in `internal/fabricvocab`; commit a85a498ee added it, and round 3 found no findings.
