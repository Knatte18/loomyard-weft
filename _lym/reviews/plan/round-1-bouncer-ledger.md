---
round: 1
ledger:
  - key: pairteardown-imports-battenrecipe-cycle
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: dobbyengine-allowlist-missing-runfsm
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: board-status-affected-set-incomplete
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: cancelled-branch-read-network-per-tick
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: shed-pause-aside-unlicensed
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: cancel-invisible-before-intent-committed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: paused-refusal-names-wrong-verb
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: drivable-reached-parent-clause-dead
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: runfsm-order-test-only
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: cancel-marker-prose-survives
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: landed-slug-pause-unliftable
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: cancel-act-rule-row-unreachable
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: nil-pairstands-meaning
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card9-impactsummary-undercount
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: loom-run-step-unasked-no-reason
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: design-wrong
---

Round 1 raised fifteen findings, none carried from an earlier round.
Two BLOCKING findings stop the plan building or passing its own gates: card 5's import cycle and card 11's import allowlist.
Three more gating design findings at MEDIUM concern a per-tick network read, a cancel window with no visible intent, and a dead `drivable` clause.
The shed-pause aside needs a parent-recorded `Added after Discussion` entry or removal.
