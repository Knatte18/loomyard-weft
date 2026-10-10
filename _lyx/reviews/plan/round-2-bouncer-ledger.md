---
round: 2
ledger:
  - key: invocable-counts-persistent-flags
    rounds: [1]
    status: resolved
  - key: fabric-clone-long-over-cap
    rounds: [1]
    status: resolved
  - key: watcher-index-binary-decision-uncarried
    rounds: [1]
    status: resolved
  - key: duplicate-audience-assertions-cards-6-8
    rounds: [1]
    status: resolved
  - key: runnable-group-print-rule
    rounds: [1]
    status: resolved
  - key: watcher-added-decisions-precedence-split
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
---

All five round 1 keys are resolved: round 2 did not re-raise them, and its focus settlements find no defect in the persistent-flag `Invocable` rule, the `Long` trim sets or the runnable-group print rule.
Round 2 opened one finding, F1: the plan settles the record's two added watcher decisions by opposite precedence rules and states only one of the conflicts.
It is a non-gating `decision` finding at MEDIUM.
