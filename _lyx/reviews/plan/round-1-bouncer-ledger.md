---
round: 1
ledger:
  - key: held-compaction-boundary-survives-stale-tick
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: plugins-step-not-carved-out-of-resuming-tick
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: loom-start-smoke-tests-assert-handover
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: pattern-cli-cobra-lists-orch-start
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: attach-probe-only-callers-inherit-teardown
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: running-turn-case-has-no-test-row
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: cards-1-to-7-name-no-module-doc
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card-8-omits-arm-go
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card-16-pins-wait-twice
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card-18-tests-verb-twice
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: submit-redraw-settle-left-open
    rounds: [1]
    status: open
    class: decision
    severity: NIT
---

Round 1, first review of the plan: eleven findings, none carried from an earlier round.
Gating are the watcher boundary finding (BLOCKING, `design`) and the plugins-step finding (MEDIUM, `design`), both about the orch watcher's reload sequencing.
The rest are missing test rows, missing doc or Edit-list entries, and two duplicated tests.
