---
round: 2
ledger:
  - key: parentless-placement-risk-false-premise
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: unused-tmux-exception-carried-into-creel
    rounds: [1]
    status: resolved
  - key: d1-preamble-shuttle-only-text
    rounds: [1]
    status: resolved
  - key: d1-powershell-7-precondition-silent
    rounds: [1]
    status: resolved
  - key: daemon-risk-misplaces-socket
    rounds: [1]
    status: resolved
  - key: shuttlesuite-teardown-criterion-redundant
    rounds: [1]
    status: resolved
  - key: creel-pane-content-observation-unstated
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: daemon-risk-omits-journal-file
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings; round 2 raised three.
The round 1 fixes cleared five keys, including the carried tmux exception, which D1 now omits.
The C13 stacked-layout risk key is reopened: the risk is now present but hedged, and its follow-up has no owner (F1, MEDIUM, gating).
Two new LOW keys remain: D1 does not say who reads pane content (F2), and the daemon risk omits the open journal file (F3).
