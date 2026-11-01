---
round: 1
ledger:
  - key: admission-process-wide
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: inflight-marker-docs
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: remove-run-records-head-read-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings, none gating.
F1 (`admission-process-wide`) is the admission's process-wide reach; F2 (`inflight-marker-docs`) is the stale NetOp and doc.go wording; F3 (`remove-run-records-head-read-untested`) is the missing test row for the failed records-HEAD read.
No prior ledger exists.
