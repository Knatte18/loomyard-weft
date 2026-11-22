---
round: 1
ledger:
  - key: sandbox-root-through-records-sibling-link
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: sandbox-guard-missing-test-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: sandbox-target-ancestor-symlinks-unresolved
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: sandbox-owner-ignores-hub
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: fabric-go-header-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: doc-go-merge-line-stale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 findings map to keys as F1, F2, F3, F4, F5 and F6 in the order above.
F1 is the gating finding: the guard resolves the target through the `.lym` link into the `-weft` sibling and so never recognises the strand's own sandbox hub.
F3 and F4 touch the same function and may be fixed together with F1.
