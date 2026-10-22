---
round: 1
ledger:
  - key: skip-list-drifts-from-scankit
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: no-sdk-dotnet-fails-instead-of-skipping
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW findings: F1 maps to the skip-list key, F2 to the no-SDK key.
Neither is gating at MEDIUM or worse.
