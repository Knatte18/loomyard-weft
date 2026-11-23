---
round: 1
ledger:
  - key: resumed-start-retry-reuses-resume
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: codex-excluded-from-every-engine
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: attach-interrupted-stop-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: orch-stencil-fresh-agent-after-resumed-halt
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: resumed-start-carried-instructions-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stopstrand-deliberate-stop-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: retire-interrupted-failure-path-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: goto-test-header-claims-unpinned-order
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: summary-blast-radius-too-narrow
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger exists.
Nine findings, all open.
Gating: F1 (`resumed-start-retry-reuses-resume`) and F3 (`attach-interrupted-stop-untested`), both design-class MEDIUM.
F2 (`codex-excluded-from-every-engine`) is a MEDIUM decision finding deferred to the run's parent.
F4 to F9 are LOW and cover a stale stencil clause, four missing test assertions or rows, a test header, and the change description's blast radius.
