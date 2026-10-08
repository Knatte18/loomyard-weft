---
round: 1
ledger:
  - key: d6-reprompt-retry-never-fires
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: stopped-outside-file-contract-first
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d1-unmatched-turn-start-holds-idle-pane
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d4-no-work-done-premise-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-settle-wait-vs-d3-window
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: stopped-overwrite-vs-burler-criterion
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: stop-verb-record-write-failure
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: acceptance-criteria-gaps
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: q8-open-while-d4-decided
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: restated-content-in-decision-record
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised ten findings, all open.
The gating ones are the D6 retry that no trigger fires (F1), the `stopped` outcome outside attach's file-contract-first check (F2), the D1 unmatched `turn_start` hold (F3) and the unbounded D4 "no work done" premise (F4).
The rest are consistency gaps between D2/D3 and D7 and the acceptance criteria, one missing failure-order decision, and a restatement NIT.
