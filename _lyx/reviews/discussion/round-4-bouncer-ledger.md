---
round: 4
ledger:
  - key: intake-mutations-unbounded-by-issue-state
    rounds: [1, 2]
    status: resolved
  - key: intake-retry-rule-underspecified
    rounds: [1, 2]
    status: resolved
  - key: fold-target-semantics-open
    rounds: [1, 2, 3]
    status: resolved
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
    status: resolved
  - key: intake-checks-not-under-lock
    rounds: [3]
    status: resolved
  - key: issues-list-unbounded-hiding
    rounds: [3, 4]
    status: open
  - key: intake-edge-cases-unstated
    rounds: [3]
    status: resolved
  - key: support-log-stale-retry-design
    rounds: [3]
    status: resolved
---

Gating counts ran 1, 2, 1, 0 across rounds 1 to 4.
Round 4 raised one LOW decision finding, F1, which maps to `issues-list-unbounded-hiding`: D9's bound omits that `upsert` of `issues` corrects a wrong hand-written number.
The review's focus settlements found the fold against D3, the lock sequencing and the close recovery consistent, and found no stale retry text in the support log.
`fold-target-semantics-open` and the other round 3 keys were not raised again and are resolved.
