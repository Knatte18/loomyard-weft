---
round: 1
ledger:
  - key: d3-commit-push-route-open
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d2-not-adopted-vs-created
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: mutation-record-retries-in-constraints
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: envelope-key-wording-d4
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d1-bound-premise-and-exhausted-step11
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: d1-classifier-mixed-stderr
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: acceptance-criteria-gaps
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: hard-cut-vs-stranded-trace
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: narrative-belongs-in-support-log
    rounds: [1]
    status: open
    class: scope
    severity: NIT
---

Round 1 is the first review of the discussion pair.
Two BLOCKING design findings stand: D3 leaves the prime's `Fabric.Commit` push route open, and D2 treats "not adopted" as "created by this call" so its rollback can delete a pre-existing origin branch.
The remaining findings are one MEDIUM consistency finding on the `PATTERN-mutation-record` constraint bullet, and LOW or NIT findings on wording, the D1 classifier and bound, missing acceptance criteria, the hard-cut assumption against stranded traces, and narrative to relocate.
All are open; none has been fixed or re-reviewed.
