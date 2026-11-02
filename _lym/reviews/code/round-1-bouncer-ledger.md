---
round: 1
ledger:
  - key: hook-batching-verb-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: transcript-fixture-no-split-message
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: transcript-tail-window-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: batching-nudge-boundary-rows-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: failure-paths-and-validation-bounds-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised five test-coverage findings against the batching nudge and its hidden verb.
Two are MEDIUM: the untested `hookBatchingCmd`, and a transcript fixture that never splits one message over several lines.
Three are LOW: the transcript tail window, boundary rows of `TestBatchingNudge`, and failure paths with validation bounds.
All five fixes are local test additions the finding text already specifies.
