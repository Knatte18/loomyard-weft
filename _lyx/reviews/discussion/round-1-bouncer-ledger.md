---
round: 1
ledger:
  - key: d9-root-cause-deferred
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: ac-hold-ending-mismatch-d9
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d2-loom-upsert-check-premise
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-unreadable-record-contract
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: d4-both-keys-migration
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d4-migrate-contract-widening
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: scope-missing-shedadapters
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: orch-stencil-change-only-in-notes
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d6-commit-history
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings against the decision record, none carried from an earlier round.
The gating pair is the D9 root cause left to Plan (BLOCKING) and D2's false premise about loom's upsert-time check (design, MEDIUM).
Three further fixes change decision text: D5's error contract for an unreadable record, D4's both-keys migration outcome, and the D2 and D1 choice about loom's upsert check.
The rest are local corrections: the criterion wording against D9, the `Migrate` doc comment note, the Scope In entry, the stencil line in D3, and the commit reference in D6.
