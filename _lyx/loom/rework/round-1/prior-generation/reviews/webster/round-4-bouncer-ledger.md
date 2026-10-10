---
round: 4
ledger:
  - key: log-level-env-reset-by-root-hook
    rounds: [1]
    status: resolved
  - key: boot-refusals-unreached-at-planned-site
    rounds: [1]
    status: resolved
  - key: cmd-lyx-test-budget-row-not-raised
    rounds: [3]
    status: resolved
---

Round 1 raised the root-hook level reset and the shadowed boot refusals; round 2 found both fixed.
Round 3 raised the unraised `cmd/lyx` test-budget row after the merge-in.
Round 4 confirmed the row at 74 and raised no findings.
