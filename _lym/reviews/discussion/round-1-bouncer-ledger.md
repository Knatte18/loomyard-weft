---
round: 1
ledger:
  - key: d7-merge-parent-lag-trigger
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d2-run-orch-agreement-down-orch
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d4-pr-review-seat-mechanism-missing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-pr-review-creel-answer-and-orch-role
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-pr-continue-regate-use
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-autonomy-toggle-mid-review
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-judge-interactive-timeout-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-raddle-auto-approve-reader
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: scope-out-task-making-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d5-class5-count-source
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-unsettleable-halt-no-route
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-drain-marker-store
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d9-change-mail-writers
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-unmarked-reply-backstop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-rationale-broken-beam-yaml-orch-hears
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d9-every-orch-pointer-overreach
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d9-beam-pr-help-texts-exception
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d9-acceptance-default-layer-write
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-files-bug-channel
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-fork-inheritance-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: auto-mode-assumption-repeats-d6
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 21 findings, none carried from an earlier round.
The two BLOCKING findings are D7 step 1's merge-parent trigger on a mere lag (review F1) and D2's run/orch agreement claim failing when the parent worktree's orch is down (F2).
Most of the rest are `design` findings on D3, D4, D5, D6, D8 and D9 about unbounded or unnamed mechanisms.
The consistency findings are local rewordings the review text already fixes.
