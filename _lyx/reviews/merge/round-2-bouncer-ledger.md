---
round: 2
ledger:
  - key: pauserecord-update-bypasses-state
    rounds: [1]
    status: resolved
  - key: state-keep-unknown-shifted-list-elements
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: pauserecord-stale-lock-import-contract
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised the parent's `pauserecord.Update` bypassing `internal/state`; the fix routed it through `state.UpdateJSON`, and round 2 found no recurrence.
Round 2 raised two new findings: `keepUnknownInList` keeps unknown keys only for list elements at an unchanged index (F1, MEDIUM design), and the pauserecord import contract still names `internal/lock` (F2, LOW consistency).
