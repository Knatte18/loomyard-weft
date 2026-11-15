---
round: 1
ledger:
  - key: gated-fragment-names-scope-form-and-go-flags
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: reproduce-run-can-skip-test-and-read-as-pass
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: spec-test-pins-gated-stencil-prose
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: reproduce-check-placed-after-edit-step
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so there is no earlier ledger.
F1 and F2 are design findings about the reproduce instruction in the gated fragment; the fix replaces it with a request to the parent that names the test and its tier.
F3 follows from F1 and removes the prose assertion in `spec_test.go`.
F4 moves the instruction ahead of the edit step.
