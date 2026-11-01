---
round: 1
ledger:
  - key: seatengine-lost-writer-end-step-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding (F1): `TestEngine_Resume_Writers` does not reach the `ender.end(lost)` branch of `Resume` for a lost writer with missing outputs or a failing Accept.
The fix is two added subtests and no production code.
