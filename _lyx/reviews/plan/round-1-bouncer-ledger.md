---
round: 1
ledger:
  - key: card5-await-startup-status-failure-exits
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card9-reed-fields-dead
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: cards8-9-rerouted-stop-bound-unstated
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: card6-boundary-idle-capability-mismatch
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card4-confirm-engine-lacks-idle-reading
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card3-duplicate-refusal-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card8-caller-miscount
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card5-abandon-startup-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card3-resume-attempts-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first round, so there is no prior ledger.
All nine findings are open and none is gating: F6 is the only design finding and is LOW.
