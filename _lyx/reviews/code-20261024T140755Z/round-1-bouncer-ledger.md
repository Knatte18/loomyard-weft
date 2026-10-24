---
round: 1
ledger:
  - key: failedtier-last-vs-first-failure
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: cancelled-context-halt-returns-green
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: halt-after-failure-drops-code-and-closing-line
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: failing-guard-run-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all `design`.
F1 (`failedtier-last-vs-first-failure`): `FailedTier` and its docs say "last" while the exit code follows the first failure.
F2 (`cancelled-context-halt-returns-green`): a halt on a cancelled context with a passing run returns 0.
F3 (`halt-after-failure-drops-code-and-closing-line`): a later halt loses the first failure's code and the closing line.
F4 (`failing-guard-run-untested`): the guard label and first-failure rule across a guard failure lack a test.
