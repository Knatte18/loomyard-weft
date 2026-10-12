---
round: 1
ledger:
  - key: overlay-invalid-json-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised one finding, F1, mapped to `overlay-invalid-json-untested`: `readOverlay`'s syntax-error arms have no test row.
It is a coverage gap only; the code refuses correctly.
No earlier ledger existed.
