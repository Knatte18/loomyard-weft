---
round: 1
ledger:
  - key: windows-tree-kill-false-premise
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: gated-strand-dropped-without-bound
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: batten-run-outcomes-unordered
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: pattern-entry-missing-for-new-invariants
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: wait-mark-stale-or-misdirected-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: revive-and-way-forward-disagree
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: goal-acceptance-overreach-scope
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: history-provenance-in-decision-record
    rounds: [1]
    status: open
    class: design
    severity: LOW
---

Round 1 is the first review of the discussion pair; all eight findings are new and open.
The two BLOCKING keys are the Windows kill premise (F1) and the gated-strand drop (F2).
The remaining design-class keys at MEDIUM are the batten run outcome ordering (F3) and the wait-mark bound (F5).
