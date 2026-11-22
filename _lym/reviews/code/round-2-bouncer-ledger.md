---
round: 2
ledger:
  - key: judge-never-reads-charter
    rounds: [1]
    status: resolved
  - key: missing-board-entry-refusal-untested
    rounds: [1]
    status: resolved
  - key: seed-brief-failure-swallowed
    rounds: [1]
    status: resolved
  - key: copies-tier-not-derived
    rounds: [1]
    status: resolved
  - key: crucible-carry-over-incomplete
    rounds: [1]
    status: resolved
  - key: fixer-rules-in-rubric-fixer-never-reads
    rounds: [1]
    status: resolved
  - key: recipe-seed-wiring-untested
    rounds: [1]
    status: resolved
  - key: preflight-tolerates-absent-gate-yaml
    rounds: [1]
    status: resolved
  - key: preflight-test-skips-template-keys
    rounds: [1]
    status: resolved
  - key: removed-checks-no-loosened-check-trailer
    rounds: [1]
    status: resolved
  - key: default-sandbox-layout-unasserted
    rounds: [1]
    status: resolved
  - key: focus-filename-redeclared
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: copies-key-retirement-no-reinterpretation
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1's eleven findings are all resolved in round 2, mapped F1 to F11 in the order of the round 1 keys.
Round 2 raises two new BLOCKING findings, both side effects of round 1's fixes.
F12 (`focus-filename-redeclared`) is the F1 fix's `focusFileFormat` constant duplicating `shedadapters.focusPath`.
F13 (`copies-key-retirement-no-reinterpretation`) is the F4 fix retiring `gates.harden-live-copies` without a config reinterpretation or a corrected landing summary.
