---
round: 1
ledger:
  - key: startedat-pinned-only-on-done-path
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: folded-stuck-entry-idle-time-unreported
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: capped-hold-apierror-and-outputs-exist-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: cap-naming-notice-text-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: nonempty-idle-gaps-envelope-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: outcomeapierror-doc-omits-capped-hold
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings (F1 to F6) in the order of the keys above.
F1 is the only gating one: StartedAt is asserted only on the done path.
F2 asks for a choice between keeping per-call gaps in a fold and documenting that they go unreported.
F3 to F6 are local test and doc gaps.
