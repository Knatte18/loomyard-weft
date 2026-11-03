---
round: 1
ledger:
  - key: renumber-insert-kept-range-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: renumber-bare-only-card-write-fault-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: renumber-index-and-heading-lines-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: glyphless-file-failure-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: loom-plan-spec-ambiguous-antecedent
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised five findings, all local corrections with their result fixed by the finding text.
F1 (MEDIUM, design) gates: the kept-range branch of `mapsFaithfully` on insert has no test row.
F2, F3 and F4 are test-coverage gaps at LOW; F5 is a spec wording NIT.
