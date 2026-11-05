---
round: 1
ledger:
  - key: runlock-exemption-passes-whole-change
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: startguard-drops-discussed-refusal
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: sketchfinish-draft-fallback-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: runlock-exemption-boundary-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: sketchfinish-fallback-on-every-error
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: sketch-missing-coverage-migration-replay-already-sketch
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: sketch-conversion-fields-under-asserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 found seven findings, none BLOCKING, four MEDIUM and three LOW.
Keys map to the review's findings as follows: F1 `runlock-exemption-passes-whole-change`, F2 `startguard-drops-discussed-refusal`, F3 `sketchfinish-draft-fallback-untested`, F6 `runlock-exemption-boundary-unpinned`, F4 `sketchfinish-fallback-on-every-error`, F5 `sketch-missing-coverage-migration-replay-already-sketch`, F7 `sketch-conversion-fields-under-asserted`.
F1 and F6 pair: F6's rows fail until F1 narrows the exemption.
