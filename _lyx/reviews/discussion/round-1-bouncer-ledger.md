---
round: 1
ledger:
  - key: d10-pair-can-write-hubwide-landing
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d7-outside-hub-unslotted-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: cli-wait-sec-vs-bash-timeout-chain
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: verifyrun-constraint-wrong-pattern-open-seam
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d2-failure-rule-no-slot-busy-carveout
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: failure-record-tamper-bound-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: acceptance-criteria-miss-decision-behavior
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 opened seven findings, all still open.
The gating ones are the hub-wide `landing` write route (F1, BLOCKING), the outside-a-hub unslotted path (F2) and the `cli_wait_sec` versus Bash timeout relation (F3).
The rest are LOW: the `verifyrun` constraint wording (F4), the slot-busy carve-out in D2 (F5), the failure record's tamper bound (F6) and uncovered acceptance criteria (F7).
