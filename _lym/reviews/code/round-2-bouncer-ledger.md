---
round: 2
ledger:
  - key: stamp-builder-untested
    rounds: [1]
    status: resolved
  - key: test-file-import-miss
    rounds: [1]
    status: resolved
  - key: stamp-miss-rows-incomplete
    rounds: [1]
    status: resolved
  - key: build-failure-beside-tests-misses
    rounds: [1]
    status: resolved
  - key: unreadable-commit-target-untested
    rounds: [1]
    status: resolved
  - key: unreadable-toolchain-miss-untested
    rounds: [1]
    status: resolved
  - key: package-clause-change-untested
    rounds: [1]
    status: resolved
  - key: summary-test-file-import-claim-stale
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: test-file-import-allowance-unrecorded
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised seven `design` findings; the round 2 review checked each against the code and its tests and found no defect, so all seven are resolved.
Round 2 raised two new MEDIUM findings, neither gating.
F1 maps to `summary-test-file-import-claim-stale`: the change description contradicts the selector on test-file imports.
F2 maps to `test-file-import-allowance-unrecorded`: the import allowance goes beyond the record's rule, and its fix is a parent correction recorded with `lym loom decision add`.
