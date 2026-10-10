---
round: 1
ledger:
  - key: card4-tmux-build-breaks-signaltick
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: cards10-11-test-budget-rows
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: card7-claudeengine-doc-false
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card8-readiness-bound-unstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card8-readiness-test-triplication
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card5-watchnames-wallclock-test
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card14-benchmark-on-live-hub
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: cards7-13-escape-hatch-bounds
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card4-default-timing-floor-new-test
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card6-discovery-cadence-undocumented
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card13-two-cmdlyx-tests
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: overview-opener-decision-vs-card6-tests
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card5-discover-signal-literal-pin
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1, first judged round: 13 findings, 2 BLOCKING and 5 MEDIUM, none recurring.
Gating are the two BLOCKING scope findings (card 4's missed integration-test file, cards 10 and 11's missed budget file) and the design-class MEDIUMs on card 8's unstated bound and card 5's wall-clock test.
Every key is open pending the fixer.
