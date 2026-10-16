---
round: 1
ledger:
  - key: d1-handon-signal-matches-d2-faults
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d1-settling-fork-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-peer-forks-unlinked-to-d1-d2
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d7-issue-673-move-misses-post-plan-landing
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: d6-bound-overstates-no-override
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d2-rationale-deliberation-narrative
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings, all still open.
The two gating ones concern D1: its hand-on signal also catches D2's verb-concluded faults, and its settling fork states no bound on what it may record or resume.
The other four are local corrections to D3, D7, D6 and D2's rationale.
