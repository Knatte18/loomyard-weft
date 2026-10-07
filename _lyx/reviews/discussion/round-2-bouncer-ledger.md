---
round: 2
ledger:
  - key: d3-commit-push-route-open
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-not-adopted-vs-created
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: mutation-record-retries-in-constraints
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: envelope-key-wording-d4
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: d1-bound-premise-and-exhausted-step11
    rounds: [1]
    status: resolved
  - key: d1-classifier-mixed-stderr
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-gaps
    rounds: [1, 2]
    status: open
    class: scope
    severity: LOW
  - key: hard-cut-vs-stranded-trace
    rounds: [1]
    status: resolved
  - key: narrative-belongs-in-support-log
    rounds: [1]
    status: resolved
  - key: d3-commit-failure-behaviour
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
---

Round 1 raised two BLOCKING design findings; round 2 shows both reduced to MEDIUM.
D3 still mis-states what `Fabric.Commit` knows (F1) and leaves its failure path unbounded (F2, new key).
D2's rollback guard still fails when the probe was skipped (F3).
The mutation-record wording (F4) and D4 rename wording (F5) persist as consistency findings.
The round-1 D1 findings, the stranded-trace assumption and the narrative relocation no longer appear in the latest review and are resolved.
Acceptance-criteria gaps persist (F6), now naming the cases D1, D2 and D3 add.
