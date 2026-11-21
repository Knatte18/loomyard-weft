---
round: 1
ledger:
  - key: embed-pattern-dir-matches-excluded-descendants
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: embedded-file-under-testdata-or-prefixed-dir-not-restored
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: embed-directive-scan-misreads-quoted-and-tab-forms
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: embeddedAtBase-read-failures-uncovered
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: loomshed-doc-stale-unjudged-cause
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: redFirstCall-run-doc-omits-embedded-files
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; there is no earlier ledger.
F1 to F6 of the review map to the keys above in order.
F1, a MEDIUM `design` finding, keeps the loop open; each other finding is LOW or NIT and each fix is spelled out in the review.
