---
round: 1
ledger:
  - key: layout-cache-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: layout-read-overlapping-clear-stale-store
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: forgetlayouts-doc-names-more-writers-than-call-it
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: layoutanswers-doc-invariant-broken-by-board-marker-retirement
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: probe-docs-call-revert-rebase-fabric-replay
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1, the first round, has no earlier ledger.
F1 is the one gating finding: the layout cache, its invalidation by fabric's marker writes and its uncached failed read have no test.
F2 is a narrow store-after-clear race, LOW.
F3 to F5 are doc-only corrections that change no code.
