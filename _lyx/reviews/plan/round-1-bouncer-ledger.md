---
round: 1
ledger:
  - key: test-budget-not-raised
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: drift-test-repeats-existing
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: reader-fallback-turn-end-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: shuttleengine-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: claudeengine-doc-missing-parser
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card-12-unlicensed-wording
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: output-files-first-no-carrier
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 opened seven findings, none resolved yet.
Mapping to review IDs: F1 `test-budget-not-raised`, F2 `drift-test-repeats-existing`, F3 `reader-fallback-turn-end-untested`, F4 `shuttleengine-doc-stale`, F5 `claudeengine-doc-missing-parser`, F6 `card-12-unlicensed-wording`, F7 `output-files-first-no-carrier`.
The two BLOCKING findings concern test economy and the per-package test budget; the rest are doc and coverage gaps.
