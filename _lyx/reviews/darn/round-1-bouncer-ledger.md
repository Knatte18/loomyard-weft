---
round: 1
ledger:
  - key: start-ledger-write-failure-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: migrate-empty-co-authored-by
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: ledger-overrecords-on-addstrand-failure
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: orphan-sweep-keeps-ledger-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: nil-enginesrun-branch-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: migrateconfig-coverage-gaps
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: describe-stencils-fixed-trailer-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: doc-comments-lean-on-other-symbols
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised eight findings (F1 to F8 in order of the keys above), all open.
F1 is the only gating one: a MEDIUM test gap on the failure path of the engine-ledger write in `Start`.
The rest are LOW test gaps, one migration edge case, one stencil wording drift and one comment-style issue.
