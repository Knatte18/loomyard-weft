---
round: 1
ledger:
  - key: remove-command-parse-test-missing
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: green-fallback-removal-line-uncovered
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1, first ledger.
F1 asks for a test that `RemoveCommand` output parses through the remove verb; F2 asks for coverage of, or removal of, the non-`RemoveError` fallback in `green.go`.
Both are local corrections whose result the finding text fixes, and neither is gating.
