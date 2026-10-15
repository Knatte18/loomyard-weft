---
round: 1
ledger:
  - key: d4-single-invocation-premise-false-for-recovery-strand
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: misspelled-skill-open-risk-contradicts-shuttle-skip
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-any-skill-allowed-without-stated-bound
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: criterion-leaves-discussion-plan-fix-rows-unpinned
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: sandbox-coverage-non-applicability-line-in-constraints
    rounds: [1]
    status: open
    class: scope
    severity: NIT
    fix: mechanical
---

Round 1 raised five findings and no earlier ledger exists.
The one gating finding is that D4 claims one list per invocation, while the Webster recovery strand is spawned by a separate `lyx webster recover-batch` process and can load a newer list than its master.
The other four are local corrections: a misspelled-skill Open risk that contradicts shuttle's skip behavior, a missing bound on D7, an acceptance criterion that omits the Discussion and Plan fix rows, and a non-applicability line in Constraints to delete.
