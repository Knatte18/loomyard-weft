---
round: 1
ledger:
  - key: perch-filing-refusal-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: raddle-run-filing-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: raddle-board-context-perch-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: store-cold-perch-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: first-mode-split-perch-error-untested
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: template-yaml-auto-approve-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stencil-heading-names-operator-only
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: rows-hold-split-perch-uncleaned-path
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 found eight findings.
Three MEDIUM `design` findings (F1 to F3) leave new `internal/loomcli` behavior without tests: the perch filing refusal's guard and error paths, `raddleRunFiling`, and `raddleBoardContext`'s `Perch`.
F4 is the same kind of gap for `StoreCold`, at LOW.
F5 to F8 are LOW or NIT consistency findings: an untested error path in `firstMode`, a stale template comment, a stale stencil heading and an uncleaned artifact path.
Every fix is a local correction its finding text already fixes.
