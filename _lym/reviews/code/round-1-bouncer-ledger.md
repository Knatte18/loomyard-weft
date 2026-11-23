---
round: 1
ledger:
  - key: build-skew-reason-misstates-driver-action
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: build-skew-classification-wider-than-entry
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: summary-claims-hand-back
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: no-row-pins-failed-state-guard
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four LOW findings.
F1 (reason wording at three sites), F3 (change description claim) and F4 (missing test row) are local corrections.
F2 is deferred to the run's parent, since recording or narrowing the wider build-skew classification is the parent's call.
