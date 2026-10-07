---
round: 4
ledger:
  - key: fabric-reference-scope-no-decision
    rounds: [1]
    status: resolved
  - key: d4-over-budget-lone-card-unbounded
    rounds: [1]
    status: resolved
  - key: d9-fit-undefined-on-thin-data
    rounds: [1]
    status: resolved
  - key: wrong-position-consequence-unstated
    rounds: [1]
    status: resolved
  - key: d11-one-profile-one-count-assumption
    rounds: [1]
    status: resolved
  - key: review-file-format-owner-unnamed
    rounds: [1]
    status: resolved
  - key: d12-rubric-paraphrases-rule
    rounds: [1]
    status: resolved
  - key: d12-wrong-enforcement-component
    rounds: [1]
    status: resolved
  - key: fork-count-disagrees-with-table
    rounds: [1]
    status: resolved
  - key: auto-mode-assumptions-restate-decisions
    rounds: [1]
    status: resolved
  - key: sed-note-belongs-nowhere
    rounds: [1]
    status: resolved
  - key: d13-shared-rule-narrower-than-audit-matcher
    rounds: [2]
    status: resolved
  - key: d14-reader-list-open-and-not-write-free
    rounds: [2]
    status: resolved
  - key: d14-ancestry-evidence-overstated
    rounds: [2]
    status: resolved
  - key: d11-multi-profile-run-contribution
    rounds: [2]
    status: resolved
  - key: d11-fresh-attempt-weight-cards-pairing
    rounds: [2]
    status: resolved
  - key: d11-card-count-source-without-partition
    rounds: [2]
    status: resolved
  - key: d13-inline-span-asymmetry-unexplained
    rounds: [2]
    status: resolved
  - key: goal-d5-guarantees-exceed-bounds
    rounds: [2]
    status: resolved
  - key: d4-report-does-not-show-over-budget-card
    rounds: [2]
    status: resolved
  - key: goal-plan-gate-guarantee-exceeds-d13-bound
    rounds: [3]
    status: resolved
  - key: d13-bound-contradicts-sibling-name-rule
    rounds: [3]
    status: resolved
  - key: constraints-omit-fabric-vocabulary
    rounds: [3]
    status: resolved
  - key: d13-exact-path-equivalence-inexact
    rounds: [3]
    status: resolved
  - key: d11-findings-per-card-denominator
    rounds: [3, 4]
    status: open
    class: decision
    severity: LOW
  - key: d5-never-over-budget-is-estimate-claim
    rounds: [3]
    status: resolved
  - key: vocabulary-constraint-omits-scan-shape
    rounds: [4]
    status: open
    class: design
    severity: MEDIUM
  - key: goal-verify-section-scope-wording
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
  - key: d13-shared-unit-understated
    rounds: [4]
    status: open
    class: design
    severity: LOW
---

Round 1 raised eleven findings, led by one BLOCKING on the fabric-reference scope; round 2 found them closed and raised nine new ones, none recurring.
Round 3 closed the round-2 keys and raised six, the gating one being the absent `PATTERN-fabric-vocabulary` Constraint.
Round 4's fresh review found the round-3 D13 wording keys (guarantee versus bound, sibling-name rule, exact-path equivalence) and the D5 estimate wording closed, and the vocabulary entry now present.
It raised four new findings.
The gating one is `vocabulary-constraint-omits-scan-shape` (`design`, MEDIUM): the entry omits that the scan is a case-insensitive substring match with a closed exempt list, which the plan writer needs to write the D8 and D13 to D15 fixtures.
The `d11-findings-per-card-denominator` key reopens at LOW: the contributing run set is unnamed and an empty denominator is undefined.
The other two are LOW: the Goal's "card's verify section" wording, and D13's shared unit being larger than three patterns.
No gating key has been open in two ledgers, so nothing circles.
