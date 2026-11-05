---
round: 1
ledger:
  - key: batten-start-guard-ignores-discussed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: setrunstatus-equal-skip-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: batten-board-write-failure-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: start-write-gated-first-row-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: discussed-list-text-layer-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: discussed-layer-render-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: status-docs-help-omit-discussed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, none recurring.
The one gating finding is that the batten start guard refuses a hand start for `draft` only, though a finished motif now carries `discussed`.
The rest are test gaps around the new status write and layer, plus help text that omits `discussed`.
