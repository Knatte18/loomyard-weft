---
round: 1
ledger:
  - key: mutant-helpers-never-switched-on-in-test
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: blockless-file-and-multi-block-attribution-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: map-literal-string-keys-held-fixed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: missing-rows-for-mutator-branches
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: var-read-only-by-another-var-never-mutated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: deleted-clause-still-evaluated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: blockless-files-default-build-context
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: object-resolution-back-on
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised eight findings, F1 to F8, none BLOCKING.
F1 to F3 are gating MEDIUM design findings: untested helpers, untested block-less parsing and multi-block attribution, and map string keys held fixed.
F4 to F7 are LOW design findings about missing test rows, a variable read only through another variable, a deleted operand still being evaluated, and the default build context.
F8 is a NIT: `parser.ParseComments` replaced `parser.SkipObjectResolution`.
