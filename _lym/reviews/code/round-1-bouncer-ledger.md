---
round: 1
ledger:
  - key: report-reading-closure-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: blank-reader-answer-no-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW test-coverage findings.
F1 maps to `report-reading-closure-untested`: the read-error line and HEAD match of the report-reading closure lack rows.
F2 maps to `blank-reader-answer-no-row`: `TestReadRubric` lacks a blank-reader row.
Neither gates, and no earlier ledger exists.
