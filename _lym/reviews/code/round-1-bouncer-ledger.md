---
round: 1
ledger:
  - key: report-lens-token-subtotal-metric
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: report-correctness-case-count
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: report-unsupported-launch-time
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW consistency findings, all in the trial report.
F1 is the lens token subtotal, which sums fresh input only where the sentence names input plus output.
F2 is the correctness lens case count, 17 where the captured report lists 18.
F3 is the launch time 11:53:07Z, which no cited record supports.
Each fix is a local text correction the finding already states.
