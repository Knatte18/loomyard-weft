---
round: 1
ledger:
  - key: wiredarn-gatefixed-choice-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: darnspec-refusals-stale-comment-no-missing-fragment-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: make-change-marker-position-row-missing
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: gated-row-asserts-static-prose
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all test coverage or test economy.
F1 (design, MEDIUM) is the only gating one: nothing tests that `wireDarn` picks the gated or ungated Step 4 prompt from the pair slug and `darn.gate_fixer`.
F2 to F4 are LOW: a stale refusals comment with no missing-fragment rows, a missing marker-position row, and extra static-prose assertions in the gated row.
Each fix is a local correction the finding text fixes, so all four are mechanical.
