---
round: 1
ledger:
  - key: d1-nonexistent-halted-state
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-held-set-omits-asking
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-no-cap-bound-false-premise
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d4-global-flag-last-argv-only
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: prerun-rerun-risk-miscounted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d4-board-wording-departure-narrative
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d2-acceptance-garbled-phrase
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
  - key: q2-stale-covered-run-list
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

First round, no prior ledger.
Eight findings are open.
The two gating ones are D1 naming a `halted` state that does not exist (the real states are `blocked` and `awaiting`) and D2's held set leaving out the `asking` reading.
The rest are a false premise behind D3's absent size cap, D4's global-flag forwarding, a miscounted pre-run risk, a deliberation sentence to delete, and two wording fixes.
