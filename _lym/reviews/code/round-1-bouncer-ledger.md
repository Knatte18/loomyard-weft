---
round: 1
ledger:
  - key: lint-findings-cut-first-cut-line-wrong
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: build-failure-report-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: lint-exit-code-and-halt-report-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: touched-files-hub-branch-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: cause-lines-dropped-when-failure-read
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: nested-module-build-path-not-worktree-relative
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: run-lints-pattern-branch-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: detached-front-replay-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: lint-verb-docs-describe-old-card-gate
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: plan-spec-tier1-lists-lints-as-steps
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised ten findings (F1 to F10, in the order of the keys above).
The four MEDIUM `design` findings gate: the lint-cut defect in the failure report and three untested behaviors.
The rest are LOW and fixed with them.
There was no earlier ledger.
