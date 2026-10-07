---
round: 5
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
    rounds: [3, 4, 5]
    status: open
    class: decision
    severity: MEDIUM
  - key: d5-never-over-budget-is-estimate-claim
    rounds: [3]
    status: resolved
  - key: vocabulary-constraint-omits-scan-shape
    rounds: [4]
    status: resolved
  - key: goal-verify-section-scope-wording
    rounds: [4]
    status: resolved
  - key: d13-shared-unit-understated
    rounds: [4]
    status: resolved
  - key: d13-preprocessing-order-misstated
    rounds: [5]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d13-markdown-span-application-undecided
    rounds: [5]
    status: open
    class: decision
    severity: MEDIUM
  - key: vocabulary-constraint-surface-list-narrow
    rounds: [5]
    status: open
    class: scope
    severity: MEDIUM
  - key: scope-out-moving-patterns-contradicts-d13
    rounds: [5]
    status: open
    class: consistency
    severity: LOW
  - key: check-over-planparser-model-lacks-raw-text
    rounds: [5]
    status: open
    class: design
    severity: LOW
  - key: acceptance-omits-contributing-run-count
    rounds: [5]
    status: open
    class: consistency
    severity: LOW
---

Rounds 1 to 3 raised twenty-six findings, all since resolved; round 4 raised four, of which the vocabulary scan-shape, Goal wording and shared-unit keys are now closed.
Round 5 raised seven, none BLOCKING.
F4 reopens `d11-findings-per-card-denominator` for a third round, now at MEDIUM as class `decision`, over the unreadable-state and no-review-file inputs; `decision` is not gating.
F1 and F2 are D13 findings: the order of preprocessing per pattern, and how the shared matcher applies per markdown span.
F3 is the vocabulary Constraint's surface list, filed as `scope`.
F6 is the only `design` finding, at LOW: the check needs raw plan text the planparser model does not hold.
No gating finding reaches MEDIUM, so the loop converges.
