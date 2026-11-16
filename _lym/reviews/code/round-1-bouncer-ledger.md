---
round: 1
ledger:
  - key: printed-covering-list-not-the-judged-pick
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: candidate-cover-keeps-unconfirmed-kill
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: unbuildable-rebuild-loop-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: guard-exclusion-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: covering-pool-no-deterministic-test
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: testplanmutants-missing-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: sourcescan-memo-call-cycle
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: sourcescan-imported-selector-matches-methods
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: timeout-key-docs-omit-redundant
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: readme-old-redundant-definition
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: mutant-timeout-bound-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: secondary-check-paths-without-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised twelve findings: F1 to F5 are gating `design` MEDIUM, the rest LOW.
F1, F2 are code defects in `internal/gatecli`: the printed covering list and a candidate-cover's unconfirmed kill.
F3, F4, F5 are test gaps for the rebuild loop, the guard exclusion and the covering pool; F5 depends on F1.
F6 to F12 are low-severity: a keep marker, two source-scan flaws, two stale docs and two further test gaps.
Each fix text fixes its result, so all are marked `mechanical`.
