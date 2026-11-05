---
round: 1
ledger:
  - key: raddle-start-clears-before-owner-check
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: clear-motif-run-closure-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: raddle-file-exemption-ignores-ended-how
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: start-clears-recipe-resolution-differs
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: sketch-finish-failure-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: dobby-done-run-refusal-doc-exception
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: loomcli-doc-ended-origin-raddle-file
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: refused-sketch-notice-upsert-payload
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first review; no earlier ledger exists.
F1 (`raddle-start-clears-before-owner-check`) is the one design-level problem: the clear must move behind the owner check and the guard's refusals.
F2 is a coverage gap on the closure that gates the clear.
F3 to F8 are local corrections or doc fixes whose text the review already fixes.
