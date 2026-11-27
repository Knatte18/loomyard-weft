---
round: 1
ledger:
  - key: test-helper-and-test-support-funcs-refused
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: group-filter-and-handle-ref-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: remaining-behaviours-and-finding-ref-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: plan-vs-schedule-section-omits-new-check
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: later-card-creating-caller-func-skipped
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: merge-danger-blast-radius-understated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 opens every key.
The gating ones are the refusal of test-helper and test-support functions and the untested classification branches.
The rest concern further test gaps, a caller created as a new function, one stale spec sentence and the change description's blast radius.
