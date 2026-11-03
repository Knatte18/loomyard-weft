---
round: 1
ledger:
  - key: clear-motif-run-commit-failure-half-removal
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: seed-child-child-recipe-write-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: clear-motif-run-error-warn-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: overview-omits-beam-create-motif-clearing
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: legacy-done-motif-run-no-backfill
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: doc-comment-cross-symbol-and-history-wording
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings, none seen before.
The gating pair is F1, where a commit failure after `removeRunRecords` deleted the run directory leaves a removal the way forward cannot finish, and F2, where the Seed-Child write of the child-recipe record has no test.
F3 to F6 are lower: missing error and warn rows, an overview gap, an unrecorded no-backfill decision for the parent, and two doc-comment wordings.
