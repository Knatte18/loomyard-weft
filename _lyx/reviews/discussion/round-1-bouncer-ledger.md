---
round: 1
ledger:
  - key: d12-commit-test-inherited-startsha
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d7-allowlist-refuses-nested-orch
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d8-reader-write-flags-unbounded
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d12-recoveries-count-not-carried
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d12-new-refusal-unmarked
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d9-gated-leftover-files
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d1-warning-no-sink-restamp-save
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d1-d2-refusals-uncovered
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: friction-constraint-no-plumbing
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: goal-stale-tally-interview-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: scope-bullets-without-criteria
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: mergeresolve-import-question-in-support-log
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d11-candidates-perishable-criterion-not-red
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d8-d5-splitter-and-pinned-tests
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

First round; no earlier ledger.
All fourteen findings of the round-1 review are open: F1 to F14 in order map to the keys above.
Three BLOCKING design findings concern D12's commit baseline, D7's allowlist and D8's reader flag bounds.
