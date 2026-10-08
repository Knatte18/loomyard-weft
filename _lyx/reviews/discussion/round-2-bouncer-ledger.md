---
round: 2
ledger:
  - key: seed-call-asks-discarded-exclusions
    rounds: [1]
    status: resolved
  - key: exclusion-union-undefined-across-fan-switch
    rounds: [1]
    status: resolved
  - key: lens-texts-restate-rubrics
    rounds: [1]
    status: resolved
  - key: acceptance-lens-test-tier-clause
    rounds: [1]
    status: resolved
  - key: fixer-start-constraint-misstated
    rounds: [1]
    status: resolved
  - key: usage-record-fan-name-no-criterion
    rounds: [1]
    status: resolved
  - key: d8-restates-listed-constraints
    rounds: [1]
    status: resolved
  - key: exclusion-eligibility-unran-lens
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: discussion-fan-additive-bias
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: rubric-change-needs-no-lens-edit
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised seven keys, one gating.
Round 2 does not raise any of them again, and its focus verdicts confirm the union arithmetic and the seed-call and fixer-start statements as sound, so all seven are resolved.
Round 2 raises three new keys.
Two are gating design findings at MEDIUM: the judge may exclude a lens that never ran (`exclusion-eligibility-unran-lens`), and the discussion fan lacks a relocation and exclusion lens while `acceptance` adds its own criterion (`discussion-fan-additive-bias`).
The third is a LOW consistency finding on the claim that a rubric change needs no lens edit.
