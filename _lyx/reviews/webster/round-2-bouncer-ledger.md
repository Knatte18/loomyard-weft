---
round: 2
ledger:
  - key: smoke-died-driver-status-refusal
    rounds: [1]
    status: resolved
  - key: smoke-malformed-status-refusal
    rounds: [1]
    status: resolved
  - key: loom-start-refusals-no-way-forward
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: gofmt-unclean-touched-files
    rounds: [1]
    status: resolved
  - key: duplicate-truncated-status-row
    rounds: [1]
    status: resolved
  - key: start-merge-in-progress-spec-row
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
---

Round 1 raised the two smoke-test refusals, the start refusals without a way forward, gofmt drift and a redundant truncated-status row.
Round 2 no longer raises the smoke refusals, the gofmt drift or the redundant row, so they are resolved.
The start-path refusals reappear in F2 with more sites, now including the merge-state probe failure and the status read on the live-driver branch.
F1 is new: `lyx loom start` reaches the `merge_in_progress` refusal, but the spec row is scoped to resume.
