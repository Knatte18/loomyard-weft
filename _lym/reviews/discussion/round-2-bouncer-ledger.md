---
round: 2
ledger:
  - key: d7-merge-parent-lag-trigger
    rounds: [1]
    status: resolved
  - key: d2-run-orch-agreement-down-orch
    rounds: [1]
    status: resolved
  - key: d4-pr-review-seat-mechanism-missing
    rounds: [1]
    status: resolved
  - key: d4-pr-review-creel-answer-and-orch-role
    rounds: [1]
    status: resolved
  - key: d4-pr-continue-regate-use
    rounds: [1]
    status: resolved
  - key: d4-autonomy-toggle-mid-review
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-judge-interactive-timeout-bound
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-raddle-auto-approve-reader
    rounds: [1]
    status: resolved
  - key: scope-out-task-making-contradiction
    rounds: [1]
    status: resolved
  - key: d5-class5-count-source
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-unsettleable-halt-no-route
    rounds: [1]
    status: resolved
  - key: d6-drain-marker-store
    rounds: [1]
    status: resolved
  - key: d9-change-mail-writers
    rounds: [1]
    status: resolved
  - key: d8-unmarked-reply-backstop
    rounds: [1]
    status: resolved
  - key: d2-rationale-broken-beam-yaml-orch-hears
    rounds: [1]
    status: resolved
  - key: d9-every-orch-pointer-overreach
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d9-beam-pr-help-texts-exception
    rounds: [1]
    status: resolved
  - key: d9-acceptance-default-layer-write
    rounds: [1]
    status: resolved
  - key: d5-files-bug-channel
    rounds: [1]
    status: resolved
  - key: d1-fork-inheritance-unpinned
    rounds: [1]
    status: resolved
  - key: auto-mode-assumption-repeats-d6
    rounds: [1]
    status: resolved
  - key: d4-pr-review-answer-verb-store
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d4-continue-flag-ends-review-unasked-head
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d4-closed-pr-reopened-by-finalize
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 raised 21 findings; round 2 resolved 17 of them and carried four forward with narrower defects.
The carried keys are the PR-Review session's autonomy recording (F26), the judge's not-sent and no-parent asks (F27), the class 5 count that ignores folded halts (F25), and the autonomy pointer path under a terminal presence (F28, the only non-gating one).
The `d9-acceptance-default-layer-write` key closed as outdated, since D9 no longer mails orchs on a write.
Round 2 added three BLOCKING `design` keys, all on D4's PR-Review autonomy path:
the answer verb writes burlerengine's run-wide ask record rather than the pull request record (F22),
the row's judge ends the review on a standing continue flag at a head no ask covered (F23),
and `pr-continue` on a closed pull request lets Finalize reopen and merge it (F24).
