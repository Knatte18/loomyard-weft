---
round: 1
ledger:
  - key: redfirst-refusals-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: merge-danger-omits-older-binary-reports
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: stored-testcover-data-left-on-disk
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: test-speed-pattern-misdescribes-testtiming
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: gateslot-doc-nontier-holder-list-empty
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: rowgate-comment-stale-rerun-mark
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: retired-gate-key-load-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised the keys above, all new.
The one gating key is `redfirst-refusals-untested`:
deleting the removed verbs' tests also deleted the only coverage of refusals the red-first verb still emits.
The remaining keys concern the change description, stale prose about the gate slot and re-run mark, and one missing load row.
