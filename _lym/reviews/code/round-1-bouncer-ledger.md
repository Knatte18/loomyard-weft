---
round: 1
ledger:
  - key: merge-danger-two-way-despite-state-key-rename
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: legacy-master-note-path-not-mapped-on-load
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: design-wrong
  - key: class-parent-write-comment-names-one-note
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: master-note-paths-doc-empty-condition
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: spawn-append-guards-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: parent-write-detail-names-one-note
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 approved the diff: the review found the allowance for every Master's friction note built for each state the new binary writes.

The open items are the merge-danger call over the renamed state.json key, the unmapped legacy key for a run in flight across the change,
two untested guards on the spawn-time append, and wording that still names a single note or overstates when the path list is empty.
