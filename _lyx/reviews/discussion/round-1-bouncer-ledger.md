---
round: 1
ledger:
  - key: fanned-fork-rules-drop-spawn-mechanics
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: fan-review-load-refusal-vs-retired-keys
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: never-mid-run-config-bound-false
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fixer-fan-wiring-undecided
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: bench-measured-scope-misstated
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d9-named-spawn-omitted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-model-change-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: engine-refusal-keys-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
---

Round 1 raised eight findings (F1 to F8), in the order of the keys above.
The blocking one is that D4 drops the spawn mechanics that `auditClusterRound` enforces.
Three design findings concern D3's load refusal, D2's false mid-run bound and the missing fixer-fan wiring decision.
The rest are consistency and statement gaps.
No earlier ledger exists, so nothing carries forward.
