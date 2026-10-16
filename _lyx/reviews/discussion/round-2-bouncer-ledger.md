---
round: 2
ledger:
  - key: fanned-fork-rules-drop-spawn-mechanics
    rounds: [1]
    status: resolved
  - key: fan-review-load-refusal-vs-retired-keys
    rounds: [1]
    status: resolved
  - key: never-mid-run-config-bound-false
    rounds: [1]
    status: resolved
  - key: fixer-fan-wiring-undecided
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: bench-measured-scope-misstated
    rounds: [1]
    status: resolved
  - key: d9-named-spawn-omitted
    rounds: [1]
    status: resolved
  - key: d3-model-change-unstated
    rounds: [1]
    status: resolved
  - key: engine-refusal-keys-unnamed
    rounds: [1]
    status: resolved
  - key: fixer-fan-way-forward-ineffective-under-pinning
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: usage-record-authoritative-without-hard-write
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: plan-lenses-starved-by-fork-call-cap
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: open-risks-omits-boilerplate-removal
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d4-bench-line-listed-as-change
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised eight findings; round 2 re-raised only the fixer-fan carrier question, now as a disagreement between D7's `fix-fan` bullet and D2's pinning (F2).
The seven other round-1 keys are resolved: the round-2 review does not raise them, and its Rejected section records the spawn-mechanics and `fan_review` loading questions as held.

Round 2 adds five keys.
The blocking one is that D7's way forward for a refused twill fixer, an empty fixer fan entry, cannot reach a round whose fans D2 already pinned (F1).
Two further design gaps concern D2's usage record, which has no hard write and no absence rule (F3), and the 6-call fork cap, which starves the Plan fan's affected-set lenses (F4).
Two mechanical findings remain: an Open risks omission (F5) and a bench line wrongly listed as a change (F6).
