---
round: 1
ledger:
  - key: coverage-build-cwd-not-module-root
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: signal-recorded-as-build-failure
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: staleness-file-set-content-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: build-timeout-grace-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: lym-build-no-waitdelay
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: named-package-no-go-file-dropped-silently
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: added-behaviours-without-test-row
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: doc-perishable-ten-second
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hang-test-real-five-second-wait
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings, none carried over.
Two design findings at MEDIUM gate the loop: the coverage build's working directory (F1) and signal handling in the lym build and context propagation (F2).
The remaining findings are test gaps, a silent skip, a missing wait bound and a doc wording.
