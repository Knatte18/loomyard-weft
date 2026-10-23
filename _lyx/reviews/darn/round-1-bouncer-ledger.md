---
round: 1
ledger:
  - key: unborn-head-nothing-stageable-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: summary-overstates-commitwritten-assertion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: directory-held-by-head-treated-as-not-held
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all first seen this round.
F1 (missing tests for the unborn-HEAD branch and the nothing-stageable outcome) maps to `unborn-head-nothing-stageable-untested`.
F2 (summary claims more than the `CommitWritten` subtest asserts) maps to `summary-overstates-commitwritten-assertion`.
F3 (a directory HEAD holds is treated as not held) maps to `directory-held-by-head-treated-as-not-held`.
None is gating at MEDIUM or worse, so the round converges.
