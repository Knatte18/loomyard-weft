---
round: 1
ledger:
  - key: d5-state-bound-not-enforced
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d5-revive-pause-race
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-enumeration-prime-orch
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: d6-outer-budget-no-mechanism
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: d6-cancellation-during-hold
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d8-reexec-ignores-hub-pause-state
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-matching-rule-two-readings
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-status-read-scope-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d3-behaviors-lack-criteria
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: restart-verb-out-bullet-deliberation
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: open-risk-not-open
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: d5-notice-dedup-not-atomic
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised twelve findings, all open.
The core problems are D5 (revive's state bound unenforced by the guarded send and notice queue, the pause race, the prime/orch enumeration), D6 (outer-budget rule and cancellation during the hold) and D8 (re-exec ignoring in-memory hub-pause handling).
The remaining LOW and NIT findings are local corrections to D3, D7, the Out bullet, the Open risks section and the notice dedup wording.
