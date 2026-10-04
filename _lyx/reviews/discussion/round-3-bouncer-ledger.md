---
round: 3
ledger:
  - key: intake-mutations-unbounded-by-issue-state
    rounds: [1, 2]
    status: resolved
  - key: intake-retry-rule-underspecified
    rounds: [1, 2]
    status: resolved
  - key: fold-target-semantics-open
    rounds: [1, 2, 3]
    status: open
  - key: migration-promise-contradicts-write-validation
    rounds: [1, 2]
    status: resolved
  - key: unknown-label-filter-hides-orphans
    rounds: [1, 2]
    status: resolved
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
    status: resolved
  - key: intake-close-recovery-wrong-reason
    rounds: [3]
    status: open
  - key: intake-checks-not-under-lock
    rounds: [3]
    status: open
  - key: issues-list-unbounded-hiding
    rounds: [3]
    status: open
  - key: intake-edge-cases-unstated
    rounds: [3]
    status: open
  - key: support-log-stale-retry-design
    rounds: [3]
    status: open
---

Round 3 raised six findings, one gating (F2, design MEDIUM); gating counts ran 1, 2, 1.
Four round 2 keys and the round 2 plan-writer-notes key were not raised again and are resolved.
`fold-target-semantics-open` recurs a third time as F1, a consistency finding on D10 fold labels against D3.
F2 is new: the `close` recovery for a failed import close ends the issue as not planned.
F3, F4, F5 and F6 are new LOW findings on D10 locking, the D9 `issues` list, two unstated edge cases, and stale support-log text.
