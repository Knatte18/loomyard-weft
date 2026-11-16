---
round: 1
ledger:
  - key: webster-fingerprint-mismatch-text-drift
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: recover-batch-way-forward-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: runplan-doc-false-no-verbs-claim
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: webster-doc-duplicates-runplan-contents
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stale-helper-names-in-beginbatch-comments
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger exists to carry forward.
All five findings are LOW or NIT, and each carries a local fix its text already specifies.
F1 and F2 are the `design`-class findings; neither reaches MEDIUM.
