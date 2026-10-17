---
round: 1
ledger:
  - key: card5-approve-reject-seed-refusal-invalid-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: card5-wirelanding-seed-decode-guard-unreachable
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card5-landingshed-doc-edit-no-anchor
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card5-runstart-header-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all against card 5 and all consistency-class.
The BLOCKING one is the approve/reject unreadable-seed refusal: its way forward is hand-editing `seed.json`, and `resolveRecipe` already refuses that input before the verb body runs.
The MEDIUM one is the same upstream rejection applied to `wireLanding`'s decode branch and its test.
The two LOW findings are a doc.go edit with no anchor and a stale `runstart.go` header.
