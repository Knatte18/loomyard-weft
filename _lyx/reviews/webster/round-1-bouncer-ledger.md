---
round: 1
ledger:
  - key: cardcovers-duplicated-batcher-websterengine
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised F1: the cover derivation exists as an unexported `cardCovers` in both `internal/batcher` and `internal/websterengine`.
The review's fix is to export it once from `internal/planparser` and delete both copies.
The rest of the range held up against the plan.
