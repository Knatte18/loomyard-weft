---
round: 1
ledger:
  - key: retagged-tier0-tests-drop-from-impact-loads
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: diff-base-ignores-args-fingerprint-module
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-relaunch-bypass-rules-mismatch-code
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: entry-gate-vs-pause-before-order
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: darn-tree-scope-verify-unaddressed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: carry-bound-omits-forged-record-and-env
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: zero-tests-failure-trips-on-run-filter
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: discovered-count-check-has-no-owner
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: partial-carry-writes-no-whole-entry
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d3-darn-lint-base-called-unchanged
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: bare-operand-drops-nonbool-refusal
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: carry-criterion-contradicts-d4-on-run
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: window-force-on-inner-open-terminal-flag
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-rationale-deliberation-narrative
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 14 findings, all open.
F1 is the one BLOCKING: the retag to tier tags hides test imports from `impactset` and `planglyph` loads, so narrowed gates skip tests.
Six MEDIUM design findings concern the diff base, `pause_before` ordering, darn's `tree_scope_verify`, the carry's forged-record bound, filtered zero-test runs and the unowned discovered-count check.
The rest are LOW or NIT consistency and bound corrections.
