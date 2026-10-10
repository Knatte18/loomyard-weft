---
round: 3
ledger:
  - key: smoke-died-driver-status-refusal
    rounds: [1]
    status: resolved
  - key: smoke-malformed-status-refusal
    rounds: [1]
    status: resolved
  - key: loom-start-refusals-no-way-forward
    rounds: [1, 2]
    status: resolved
  - key: gofmt-unclean-touched-files
    rounds: [1]
    status: resolved
  - key: duplicate-truncated-status-row
    rounds: [1]
    status: resolved
  - key: start-merge-in-progress-spec-row
    rounds: [2]
    status: resolved
  - key: recorded-parent-branch-reader-duplicated
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Rounds 1 and 2 raised the smoke-test status refusals, the start refusals without a way forward, gofmt drift, a redundant truncated-status row and the missing `start` scope on the merge-in-progress spec row.
Round 3 no longer raises any of them: the review checks the start refusals, the spec row and the Publish-failure tests and finds them sound, so all are resolved.
The one open key is F1, a LOW duplication of the `ParentBranch` closure's read in `readRecordedParentBranch`.
