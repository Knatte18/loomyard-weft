---
round: 1
ledger:
  - key: accepted-not-bound-to-head
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pending-fix-review-resume-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: spec-old-halt-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: accept-path-coverage-gaps
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: tapestry-no-reroute-assertion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: tests-model-finalize-reroute-cap
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: overview-circling-accept-bouncer-only
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: duplicate-resume-coverage-spent-cap
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised eight findings.
The two gating ones are F1 (`accepted-not-bound-to-head`, the fix needs a new `AcceptedHead` contract and so a fresh review) and F2 (`pending-fix-review-resume-untested`, test rows only).
The rest are LOW or NIT coverage and wording corrections.
