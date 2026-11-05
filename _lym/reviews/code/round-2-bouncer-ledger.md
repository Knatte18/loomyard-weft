---
round: 2
ledger:
  - key: runlock-exemption-passes-whole-change
    rounds: [1]
    status: resolved
  - key: startguard-drops-discussed-refusal
    rounds: [1]
    status: resolved
  - key: sketchfinish-draft-fallback-untested
    rounds: [1]
    status: resolved
  - key: runlock-exemption-boundary-unpinned
    rounds: [1]
    status: resolved
  - key: sketchfinish-fallback-on-every-error
    rounds: [1]
    status: resolved
  - key: sketch-missing-coverage-migration-replay-already-sketch
    rounds: [1]
    status: resolved
  - key: sketch-conversion-fields-under-asserted
    rounds: [1]
    status: resolved
  - key: sketch-doc-says-dates-stay-but-revised-restamped
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: sketch-exemption-retained-status-row-missing
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: summary-claims-fallback-untested
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, four MEDIUM design.
Round 2 found them fixed: the review's four focus points came back as not-a-defect.
It raised three new LOW findings, none gating: F1 `sketch-doc-says-dates-stay-but-revised-restamped`, F2 `sketch-exemption-retained-status-row-missing`, F3 `summary-claims-fallback-untested`.
