---
round: 3
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
    status: open
    class: consistency
    severity: MEDIUM
  - key: d13-bound-contradicts-sibling-name-rule
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: constraints-omit-fabric-vocabulary
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d13-exact-path-equivalence-inexact
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: d11-findings-per-card-denominator
    rounds: [3]
    status: open
    class: decision
    severity: LOW
  - key: d5-never-over-budget-is-estimate-claim
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised eleven findings, led by one BLOCKING on the fabric-reference scope; round 2 found all of them closed and raised nine new ones, none recurring.
Round 3's fresh review found the round-2 keys closed (reader list, ancestry evidence, fresh-attempt pairing, card-count source, inline-span reason, D4 report), and settled the D11 multi-profile rule in principle.
The round-2 residue shows up as new keys, not recurrences: D13's bound wording (F1, F2, F4) and the D11 denominator (F5), plus the D5 estimate wording (F6).
The one gating finding is `constraints-omit-fabric-vocabulary` (`design`, MEDIUM): Constraints name no `PATTERN-fabric-vocabulary`, though D13 to D15 put the weft and warp tokens where its scan looks.
No key has been open in two ledgers, so nothing circles.
