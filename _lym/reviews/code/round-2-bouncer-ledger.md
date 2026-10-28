---
round: 2
ledger:
  - key: output-above-header-ignores-package
    rounds: [1, 2]
    status: resolved
  - key: retry-wiring-uncovered
    rounds: [1, 2]
    status: resolved
  - key: failures-first-entry-unbounded
    rounds: [1, 2]
    status: resolved
  - key: acceptance-criteria-partly-asserted
    rounds: [1, 2]
    status: resolved
  - key: retry-delay-cancel-untested
    rounds: [1, 2]
    status: resolved
---

Round 1 raised five `design` findings, two MEDIUM and three LOW.
Round 2 found no defects, and its rejected focus items show each earlier finding addressed, so all five keys are resolved.
