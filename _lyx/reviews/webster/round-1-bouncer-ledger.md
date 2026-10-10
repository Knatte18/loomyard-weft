---
round: 1
ledger:
  - key: smoke-died-driver-status-refusal
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: smoke-malformed-status-refusal
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: loom-start-refusals-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: gofmt-unclean-touched-files
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: duplicate-truncated-status-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
---

Round 1 maps F1 and F2 to the two Publish-failure smoke rows, F3 to the two refusals without a way forward or spec row, F4 to the gofmt drift in two files, and F5 to the redundant truncated-status test row.
