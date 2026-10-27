---
round: 1
ledger:
  - key: prepare-creel-hook-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: daemon-start-reconcile-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: notice-reservation-failure-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: mail-flag-return-row-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: claim-budget-ignores-joining-newlines
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised five findings, four LOW coverage gaps (F1-F4) and one NIT on the claim budget (F5).
Each fix is a local test row or a one-line count change the finding text already specifies.
