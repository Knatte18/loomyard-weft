---
round: 2
ledger:
  - key: end-signals-reused-pid
    rounds: [1]
    status: resolved
  - key: remove-gone-worktree-skips-owner-guard
    rounds: [1]
    status: resolved
  - key: gated-arrival-ignores-missing-outputs
    rounds: [1]
    status: resolved
  - key: marker-pass-kills-merging-step
    rounds: [1]
    status: resolved
  - key: paused-parent-lock-way-forward
    rounds: [1]
    status: resolved
  - key: unknown-holder-way-forward-circular
    rounds: [1]
    status: resolved
  - key: webster-doc-old-shell-rule
    rounds: [1]
    status: resolved
  - key: register-lock-file-unspecified
    rounds: [1]
    status: resolved
  - key: stopped-refusal-live-pair-retry
    rounds: [1]
    status: resolved
  - key: owned-elsewhere-warn-no-next-step
    rounds: [1]
    status: resolved
  - key: reed-down-help-not-updated
    rounds: [1]
    status: resolved
  - key: card17-omits-broken-shell-wait-tests
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card17-stale-shell-wait-comments
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: gone-branch-remove-refusal-loops
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: live-parent-branch-refusal-unreachable
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: end-identity-check-rows-missing
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: run-lock-holder-passes-unused-input
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: unpaused-run-lock-refusal-retry-never-succeeds
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised eleven findings, none BLOCKING; round 2 did not raise any of them again, so all are resolved.
Round 2 raised seven new findings.
The BLOCKING one is card 17 leaving three existing tests, which pin the old shell-wait rule, out of its targets.
Card 17 also leaves four stale comments and docs of that rule untouched.
In card 11, a gone-origin-branch stop refuses with a way forward that names the refused `lyx fabric remove`, and the live-parent-branch refusal has no input that reaches it.
Card 2's `TestEnd` lacks rows for the group identity check and the SIGKILL-phase pid reuse of a recorded non-leader.
Card 10's `runLockHolderPasses` takes an input it never reads, and its unpaused refusal names a retry that cannot succeed.
