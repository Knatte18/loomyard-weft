---
round: 1
ledger:
  - key: jacquard-rewrite-test-rows-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: anchored-jacquard-block-dangling-alias
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: merge-key-default-half-dropped
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: duplicate-jacquard-default-assertion
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 approved the diff and raised only edge-path gaps in the jacquard reinterpretation of `internal/runprofile` and its tests.
The entries stay open as the review states them; this judge pass does not check any fix.
