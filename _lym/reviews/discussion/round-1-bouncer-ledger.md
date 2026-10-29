---
round: 1
ledger:
  - key: mode-check-research-deferred
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: operator-stop-copy-clamped-5-lines
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: version-risk-no-trigger
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: ring-wake-operator-copy-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: ring-answer-path-early-exit-release
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: ack-journaled-before-confirmation
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: only-overstates-ring-answered
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings, none carried from an earlier round.
Two are gating: the mode-check research left to the smoke run (F1) and the operator's Stop copy clamped to 5 lines once `systemMessage` is dropped (F2).
The rest are LOW gaps in the ring path's stated bounds (F3 to F6) and one wording NIT (F7).
