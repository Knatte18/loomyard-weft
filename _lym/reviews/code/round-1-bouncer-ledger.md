---
round: 1
ledger:
  - key: reed-window-help-old-title-forms
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: statuswindowtitle-comment-old-example
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: window-test-stub-title-old-format
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: title-appends-empty-name
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised four findings, none gating.
F1 to F3 are stale old-format title strings in help text, a doc comment and a test stub; F4 is an empty-name boundary slip in `Title`.
Each fix is a local correction the finding text already fixes.
