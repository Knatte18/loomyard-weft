---
round: 4
ledger:
  - key: rebaseline-refusals-untested
    rounds: [1]
    status: resolved
  - key: edited-done-cards-duplicate-gate-row
    rounds: [1, 2]
    status: resolved
  - key: ungated-waiting-empty-list-not-at-once
    rounds: [1]
    status: resolved
  - key: loop-agrees-stale-waited-out-comment
    rounds: [1, 2]
    status: resolved
  - key: touched-files-not-gofmt-clean
    rounds: [2]
    status: resolved
  - key: waiting-done-row-duplicates-background-table
    rounds: [2]
    status: resolved
  - key: loomshed-test-build-newplangate-arity
    rounds: [3]
    status: resolved
---

Round 4's review raised no findings.
The one open key from round 3, the `NewPlanGate` arity mismatch in `internal/loomshed` tests, is fixed and carried as resolved.
Every key from rounds 1 to 3 is now resolved.
