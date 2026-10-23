---
round: 1
ledger:
  - key: merge-in-align-from-lost-on-error-stop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: spent-cap-non-spent-end-no-resume
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: undecodable-attempt-record-halt-unmarked
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d5-rubric-content-uncarried
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card-51-namepr-gate-test-readers
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card-13-darn-recipe-seam-list
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card-26-code-review-off-nil-default
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card-21-23-retired-keys-location
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card-15-round-scenario-failure-rows
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: landingshed-doc-require-pr-paragraph
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: shed-recipe-spec-reject-review-name
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card-11-timeout-row-no-clock
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card-29-duplicate-verified-tree-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so there is no earlier ledger to carry forward.
Three design-class MEDIUM findings gate: the lost `AlignFrom` fork point at Merge-In, the undefined resume after a non-spent end on the last fixer turn, and the unmarked undecodable-attempt-record halt.
The first two change an approach and need a fresh review of the fix; the third is a local prefix correction.
The remaining findings are scope or consistency corrections and one LOW design note, none gating.
