---
round: 1
ledger:
  - key: parent-directive-pattern-stale-second-interactive-role
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: verbstatus-operator-session-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: recorded-mode-from-second-config-read
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: seats-header-says-non-interactive
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: resumed-live-agent-mode-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: autonomous-verify-text-not-as-before
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: operator-ruled-and-withdrawn-acceptance-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, none gating.
Two are MEDIUM: the stale `PATTERN-parent-directive` entry and the untested `verbStatus` operator mark.
The rest are LOW: the mode recorded from a second config read, the stale `seats.go` header, the untested resume path, the autonomous text wording against the change description, and the operator-ruled acceptance that awaits the parent's decision.
