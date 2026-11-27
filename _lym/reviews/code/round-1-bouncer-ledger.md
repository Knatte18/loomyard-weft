---
round: 1
ledger:
  - key: scratch-files-alone-claim-interrupted-spawn
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: writer-stencil-asserts-commit-ownership
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: darndeps-prior-work-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: merge-danger-blast-radius-short
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: recorded-outcome-hides-later-interrupted-spawn
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: scratch-walk-error-halts-darn-write
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: scratch-listing-cap-drops-top-level-files
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: darninputs-priorwork-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: scratch-list-after-recorded-outcome-unpinned
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 opened every key above.
The review reports that the diff does what the board entry asks.
The open points are a false interrupted note when only scratch files exist, a writer stencil sentence stronger than the new note,
untested production wiring, and smaller edges in the scratch listing, docs, tests and change description.
