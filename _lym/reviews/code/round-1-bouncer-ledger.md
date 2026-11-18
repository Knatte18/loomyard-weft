---
round: 1
ledger:
  - key: step-rewrite-changes-constant-expression-value
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: build-round-cap-whole-file-switch-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: embedded-line-separator-precedence-and-blank-fallback-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: mutant-build-rounds-doc-off-by-one
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings.
F1 maps to `step-rewrite-changes-constant-expression-value`: the integer-constant step rewrite makes constant expressions non-constant, changing values such as `(1 << 63) / 2`.
F2 maps to `build-round-cap-whole-file-switch-untested`: no test reaches round `mutantBuildRounds`.
F3 maps to `embedded-line-separator-precedence-and-blank-fallback-untested`.
F4 maps to `mutant-build-rounds-doc-off-by-one`.
