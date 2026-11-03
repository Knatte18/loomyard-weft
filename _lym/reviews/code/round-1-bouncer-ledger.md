---
round: 1
ledger:
  - key: scope-replacement-loses-capture-skip-list
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: anchored-start-dir-not-pinned-at-wiring
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failed-capture-removal-leaves-scope-replaced
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: gateslot-holder-worktree-became-anchor
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: non-breakaway-start-error-offered-defer
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 findings, mapped one to one: F1 to `scope-replacement-loses-capture-skip-list`, F2 to `anchored-start-dir-not-pinned-at-wiring`, F3 to `failed-capture-removal-leaves-scope-replaced`, F4 to `gateslot-holder-worktree-became-anchor`, F5 to `non-breakaway-start-error-offered-defer`.
F1 changes how `RecordScope` decides what a capture skipped, so its fix needs a fresh review.
The others are local corrections the finding text already fixes.
