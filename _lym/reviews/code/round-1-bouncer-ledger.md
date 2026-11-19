---
round: 1
ledger:
  - key: merge-danger-scan-ignores-fences
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: glossary-lines-carry-implementation-detail
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: beamcli-draft-help-doc-wayforward-miss-placeholders
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: untested-labels-directly-under-heading
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: untested-level-one-heading-ends-section
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: untested-door-final-period-strip
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: untested-refusal-of-unedited-beam-draft
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: discussion-glossary-value-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: shipped-entry-row-pins-glossary-heading-words
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: glossary-heading-swallows-stencil-section
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

First round, no earlier ledger.
F1 (fenced examples in the Merge danger scan) is the one gating finding.
F4 to F8 are test gaps the mutation report surfaced, each fixed by one table row or assertion.
F2, F3, F9 and F10 are wording and consistency corrections.
