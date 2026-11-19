---
round: 1
ledger:
  - key: stray-test-windows-runtime-skip
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: copies-run-drops-stray-failure-from-report
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: rerun-treats-first-attempt-stray-files-as-present
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: refusal-skip-has-no-test-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stray-file-names-not-escaped
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: truncation-row-repeats-code-slice
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first round and there is no earlier ledger.
F1 is the PATTERN-test-speed violation in the new integration test file.
F2 and F3 are gating design gaps in the stray-file check: the copies failure report and the per-attempt listing.
F4 to F6 are LOW test and formatting gaps.
