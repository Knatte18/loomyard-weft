---
round: 1
ledger:
  - key: dotfill-cross-client-test-geometry
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: driverbootstrap-stub-color-line
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: stale-statusline-comments-lifecycle
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: burlercli-wiring-test-reponame-header
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
  - key: reedengine-doc-strand-options-reassert
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first review of the branch.
Two `tmux`-tier tests fail on the branch and pass on `main`: the dot-fill cross-client control (F1) and the driver-bootstrap stub (F2).
Three comment and doc inaccuracies remain after card 6 and card 3 (F3, F4, F5).
All five keys are open until a fresh review sees the fixed text.
