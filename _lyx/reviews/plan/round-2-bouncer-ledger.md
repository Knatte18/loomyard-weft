---
round: 2
ledger:
  - key: card20-coverage-citation-unsupported
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: card17-orch-stencil-check-outcome-unstated
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: card21-watcher-table-underspecified
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card20-skillunknown-live-path-dropped
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: card21-startup-fold-row-shape-missing
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: card20-fakes-target-and-impact-inaccurate
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card17-changed-facts-unpinned
    rounds: [2]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised three findings; round 2 raised seven, and all three round-1 keys are mapped again.
Card 20 still mis-cites coverage for the deleted `Runner` delegation tests (F1), and a second gap shows that `Runner.SkillUnknown` loses its live-path test (F2).
Card 21's table rows are underspecified for both folds (F3, F4).
Card 17 leaves the After landing outcome unstated (F6) and pins too few of its changed facts (F7).
Card 20's `fakes_test.go` target and impact line are inaccurate (F5).
