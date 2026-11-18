---
round: 1
ledger:
  - key: refusal-rejects-valid-unkeyed-addition
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: sealed-expectation-stops-parsing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: decision-exemption-passes-unwritten-edits
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: decision-add-forms-still-halt-judge
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: drift-deps-decision-record-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: exemption-fail-closed-rows-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, all first seen this round.
F1 (the refusal on ranges over fully keyed cases) is the BLOCKING one, and F2 is settled by removing that refusal.
F3 and F4 concern how far the decision-record exemption reaches in the judge snapshot.
F5 and F6 are test-coverage gaps around the same exemption and its wiring.
