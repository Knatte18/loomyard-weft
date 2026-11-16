---
round: 1
ledger:
  - key: retro-sh-inline-lym-paths
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: future-until-skips-runs
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fileretro-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: plan-taskfields-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: median-figure-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: cap-tie-loses-run
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: duplicate-slug-way-forward
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: named-run-from-contradicts-stencil
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: parsewindow-success-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: background-gate-failure-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: gate-fixer-column-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: named-lookback-refusal-misstates
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: summary-overclaims-test
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: other-engine-transcripts-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first review: F1 to F14 map to the keys above in order.
F1 is a gating PATTERN violation; F2 to F5 are design-class MEDIUMs.
F2 is marked design-wrong because its fix chooses between a refusal and a clamp, which changes the command's contract.
The rest are local corrections or added tests whose text the finding fixes.
