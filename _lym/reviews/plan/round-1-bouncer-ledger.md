---
round: 1
ledger:
  - key: reentry-skips-card-gates
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: design-wrong
  - key: verify-gate-category-none
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: conflict-record-rel-test-only
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: run-wiring-unlisted
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: stoprecord-never-cleared
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fixer-record-not-terminal
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: delivered-overwritten-by-save
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reset-commit-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: predicates-missing-resign-arrows
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: per-helper-tests-duplicate-testrun
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failuretrace-reads-webster-only
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: conflict-path-affected-set-incomplete
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: verify-site-docs-webster-only
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card23-corrupt-row-unreachable
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: done-commit-failure-untested
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: lymdirs-doc-weaverdirname
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: extends-targets-rewrite-content
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: weaver-slot-name-repeated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first review, so every key is new and open.
The three BLOCKING findings are the missing card gates on the no-spawn re-entry path (F1), the `verify_gate` key classified `CategoryNone` (F2), and a production function only a test-only oracle calls (F3).
Design-class MEDIUMs cover Run wiring, StopRecord lifetime, delivered-heading loss, reset's commit bound and per-helper test economy.
The rest are consistency and scope items with mechanical fixes.
