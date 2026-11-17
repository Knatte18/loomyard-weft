---
round: 1
ledger:
  - key: kept-cancel-intent-written-before-commit-can-go
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: orphan-first-sight-recorded-before-notice-posts
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: handed-over-removal-skips-reed-session-end
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: tear-down-leftover-dispatch-and-workspace-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: orphan-and-handed-over-removals-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: dobby-orphans-classification-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: mark-kept-cancels-selection-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: dobby-lines-kept-row-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pair-push-deferred-reading-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: clear-done-runs-pair-skip-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pair-last-commit-error-fails-whole-orphan-listing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: remove-orphan-pair-rechecks-nothing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: pair-last-commit-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: dobby-runs-new-fields-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: orphans-failure-path-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: dobbyengine-doc-kept-bullet-splits-runs-bullet
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pairteardown-doc-omits-new-removals
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: tender-resume-help-overlong-line
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 18 findings, F1 to F18, in the order of the keys above.
Ten are MEDIUM `design` and gate: three correctness defects (F1 to F3) and seven coverage gaps against D7's criteria (F4 to F10).
The LOW and NIT findings are a listing-robustness gap, a missing re-check before an orphan removal, four test gaps, and three doc or help-text consistency items.
