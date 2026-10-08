---
round: 2
ledger:
  - key: comment-break-at-non-clause-comma
    rounds: [1]
    status: resolved
  - key: pushpairanchored-doc-old-side-names
    rounds: [1]
    status: resolved
  - key: package-doc-old-field-names
    rounds: [1]
    status: resolved
  - key: old-function-names-in-tests
    rounds: [1]
    status: resolved
  - key: fabricengine-budget-row-off-by-one
    rounds: [1]
    status: resolved
  - key: new-gate-refusals-untested
    rounds: [1]
    status: resolved
  - key: pair-walk-test-doc-omits-prime-subtests
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: mixed-side-words-in-test-comment
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

All six round 1 findings are resolved: round 2's fresh review checked the comment breaks, rename residue, budget row and gate coverage and raised none of them again.
Round 2 raised two new consistency findings.
F1 is the pair-walk push test's doc omitting the two prime subtests.
F2 is a test comment naming one side as both `code` and `warp`.
Neither is gating.
