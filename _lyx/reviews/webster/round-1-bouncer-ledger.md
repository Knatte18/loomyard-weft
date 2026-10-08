---
round: 1
ledger:
  - key: comment-fixed-column-wraps
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: redundant-package-target-unit-as-path
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: spec-paragraph-multi-sentence-lines
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: duplicate-findings-repeated-target
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: interface-method-func-prefix-detection
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: scan-directory-skips-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: package-overview-omits-plangatepass
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: test-row-pins-unreachable-input
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: repeated-plan-gate-assertion
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised nine findings: F1 to F9 in the review, in the order of the keys above.
The two BLOCKING ones are the comment wraps (F1) and the `Glyph.Unit` read as a disk path in the redundancy check (F2).
All nine are open until a fresh reviewer re-checks the fixed text.
