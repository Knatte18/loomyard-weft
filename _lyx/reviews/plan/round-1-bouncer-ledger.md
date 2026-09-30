---
round: 1
ledger:
  - key: transient-classes-unreachable-landing-arms-stuck
    rounds: [1]
    status: resolved
  - key: bootstrap-push-claim-false
    rounds: [1]
    status: resolved
  - key: gitexec-subcommand-misreads-dash-c
    rounds: [1]
    status: resolved
  - key: resume-loop-resends-on-any-error
    rounds: [1]
    status: resolved
  - key: github-deadline-not-transient
    rounds: [1]
    status: resolved
  - key: pull-beyond-d2-op-list
    rounds: [1]
    status: resolved
  - key: loom-step-help-missing-transient
    rounds: [1]
    status: resolved
  - key: overview-last-step-sentence-stale
    rounds: [1]
    status: resolved
  - key: card1-impactsummary-wrong-card
    rounds: [1]
    status: resolved
  - key: card2-bundles-two-classifiers
    rounds: [1]
    status: resolved
  - key: card9-bundles-driver-resume-line
    rounds: [1]
    status: resolved
  - key: batten-prestep-mark-untested
    rounds: [1]
    status: resolved
  - key: batten-end-to-end-criterion-unmapped
    rounds: [1]
    status: resolved
  - key: clean-tree-guard-extra-path-filter
    rounds: [1]
    status: resolved
  - key: publish-approval-query-arm-not-transient
    rounds: [1]
    status: open
  - key: transient-error-branch-bypasses-cancellation
    rounds: [1]
    status: open
---

Round 1 reviewed an 11-card plan and raised fourteen findings.
Three of them were BLOCKING: the landing producers turned every push and GitHub failure into `Stuck`, card 6 claimed a bootstrap push that never happens, and the `-c` global option was read as the git subcommand.
The fixer rewrote the plan as 14 cards, and every one of the fourteen findings checks out as fixed against the code.
The fixer narrowed F1 by keeping batten's Worktree-Create as a verdict, because a rolled-back create leaves its branch behind; the judge ratified that.

Two findings remain open against the revised card 8.
It misses `Publish.checkApproval`'s `FindPullRequest` arm, which the approve-then-start path reaches first; this one is BLOCKING.
And its new classified-error branch has to call `cancelErr` before it returns, because the push, query and create arms rely on `stuckOrCancelled` for that check (LOW).
