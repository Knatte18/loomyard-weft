---
round: 1
ledger:
  - key: parentless-placement-risk-false-premise
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: unused-tmux-exception-carried-into-creel
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-preamble-shuttle-only-text
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-powershell-7-precondition-silent
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: daemon-risk-misplaces-socket
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: shuttlesuite-teardown-criterion-redundant
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, none seen before.
Two are gating design findings at MEDIUM: the parentless-placement open risk misstates what the sibling task sweeps and misses the C13 stacked-layout risk, and D1 carries a shuttle tmux exception no creel scenario uses and that has no applicable bound.
The four LOW findings are a stale shuttle-only preamble line, a silent PowerShell 7 pre-condition, a misplaced socket in the daemon risk and a redundant teardown criterion.
