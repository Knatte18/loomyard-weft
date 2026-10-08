---
round: 1
ledger:
  - key: shadow-refresh-misses-late-markers
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: comments-cite-plan-artifacts
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: session-state-test-not-parallel
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised three findings.
The shadow's fact refresh passes only the tick's new signals, so the interrupt and API-error markers are missed when they land after the turn start or turn end was folded (F1).
Two comments cite plan artifacts instead of stable identifiers (F2).
`TestRunner_SessionState` is serial without a stated reason (F3).
