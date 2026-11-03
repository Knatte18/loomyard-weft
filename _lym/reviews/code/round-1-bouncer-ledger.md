---
round: 1
ledger:
  - key: finalize-rule-read-failure-direct-push
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reviewended-open-pr-wait-forever
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: rule-read-token-and-cancel-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: rule-read-skip-conditions-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all `design`.
F1 (`finalize-rule-read-failure-direct-push`) changes Finalize's failure behavior to a halt when a record exists, so it needs a fresh review.
F2 (`reviewended-open-pr-wait-forever`) adds an open-case verdict to `judge` that mirrors the closed case.
F3 (`rule-read-token-and-cancel-untested`) and F4 (`rule-read-skip-conditions-untested`) are test rows only.
