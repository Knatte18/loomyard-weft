---
round: 1
ledger:
  - key: d6-approval-record-and-moved-head-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d3-timeout-failed-start-reprompt-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d2-spent-fixer-no-review-weakly-pinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d5-fix-review-model-spec-and-timeout-unasserted
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: dirtied-tree-halt-contradictory-way-forward
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: fixer-key-docs-not-widened-to-fix-review
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 (first round, no prior ledger) raised six findings.
F1 to F4 are test gaps for D6, D3, D2 and D5 of the over-cap fixer review.
F5 is a halt message whose tail contradicts its own way forward.
F6 is two docs that still scope the fixer keys to the merge fixer alone.
Every fix is local and its result is fixed by the finding text, so all six are `mechanical`.
