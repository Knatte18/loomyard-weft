---
round: 2
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
    status: open
    class: decision
    severity: MEDIUM
  - key: d14-reader-list-open-and-not-write-free
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d14-ancestry-evidence-overstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d11-multi-profile-run-contribution
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: d11-fresh-attempt-weight-cards-pairing
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: d11-card-count-source-without-partition
    rounds: [2]
    status: open
    class: decision
    severity: LOW
  - key: d13-inline-span-asymmetry-unexplained
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: goal-d5-guarantees-exceed-bounds
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d4-report-does-not-show-over-budget-card
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised eleven findings, led by one BLOCKING: the folded-in fabric-reference scope had no decisions, criteria or bounds.
Round 2's fresh review raised none of those keys again, so all round-1 keys are carried as resolved; the fabric scope is now D13 to D15.
Round 2 found nine new findings, none recurring.
The gating one is the open, write-capable reader list in D14 (`design`, MEDIUM).
Three further MEDIUM `decision` findings are open: D13's shared matching rule has no importable home and is narrower than the audit's matcher, and D11 leaves two cases unsettled (a multi-profile run, and a `--fresh` run's attempt pairing).
The five LOW findings are wording and source gaps in D4, D5, D11, D13 and D14.
