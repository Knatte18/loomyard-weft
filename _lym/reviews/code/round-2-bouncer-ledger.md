---
round: 2
ledger:
  - key: retro-sh-inline-lym-paths
    rounds: [1]
    status: resolved
  - key: future-until-skips-runs
    rounds: [1]
    status: resolved
  - key: fileretro-untested
    rounds: [1]
    status: resolved
  - key: plan-taskfields-untested
    rounds: [1]
    status: resolved
  - key: median-figure-unpinned
    rounds: [1]
    status: resolved
  - key: cap-tie-loses-run
    rounds: [1]
    status: resolved
  - key: duplicate-slug-way-forward
    rounds: [1]
    status: resolved
  - key: named-run-from-contradicts-stencil
    rounds: [1]
    status: resolved
  - key: parsewindow-success-untested
    rounds: [1]
    status: resolved
  - key: background-gate-failure-untested
    rounds: [1]
    status: resolved
  - key: gate-fixer-column-unpinned
    rounds: [1]
    status: resolved
  - key: named-lookback-refusal-misstates
    rounds: [1]
    status: resolved
  - key: summary-overclaims-test
    rounds: [1]
    status: resolved
  - key: other-engine-transcripts-missing
    rounds: [1]
    status: resolved
  - key: select-widening-panics
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: retro-test-inline-lym-paths
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: outlier-fixture-or-and-unpinned
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: summary-tests-list-stale
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stencil-change-nothing-else
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised 14 findings; round 2 reports each of those keys covered or not a defect, so all are resolved.
Round 2 raises five new keys: F1 `select-widening-panics`, F2 `retro-test-inline-lym-paths`, F3 `outlier-fixture-or-and-unpinned`,
F4 `summary-tests-list-stale` and F5 `stencil-change-nothing-else`.
F1 is a defect in the widening loop that the round 1 `cap-tie-loses-run` fix added.
F1 and F2 are BLOCKING, and all five carry mechanical fixes the findings spell out.
