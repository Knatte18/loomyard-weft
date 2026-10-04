---
round: 2
ledger:
  - key: intake-mutations-unbounded-by-issue-state
    rounds: [1, 2]
    status: open
  - key: intake-retry-rule-underspecified
    rounds: [1, 2]
    status: open
  - key: fold-target-semantics-open
    rounds: [1, 2]
    status: open
  - key: migration-promise-contradicts-write-validation
    rounds: [1, 2]
    status: open
  - key: unknown-label-filter-hides-orphans
    rounds: [1, 2]
    status: open
  - key: migration-label-edge-cases
    rounds: [1]
    status: resolved
  - key: body-file-read-bound
    rounds: [1]
    status: resolved
  - key: issues-write-semantics
    rounds: [1]
    status: resolved
  - key: notes-list-importers-and-tally
    rounds: [2]
    status: open
---

Round 1 raised eight findings; round 2 raised six, two of them gating design findings.
Three round 1 keys were not raised again and are resolved.
Five reappear in new form on the fixed text: the D10 intake bound (F1 import collisions, F2 retry scope), the D10 fold (F4 labels), the D12 rationale (F3), and D6 (F5).
F6 is new: the plan-writer notes list importers and a tally.
Gating design findings went from 1 to 2, both on D10.
