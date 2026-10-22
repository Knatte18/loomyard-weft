---
round: 3
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
    rounds: [2, 3]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card17-stale-shell-wait-comments
    rounds: [2]
    status: resolved
  - key: gone-branch-remove-refusal-loops
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: live-parent-branch-refusal-unreachable
    rounds: [2]
    status: resolved
  - key: end-identity-check-rows-missing
    rounds: [2, 3]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: run-lock-holder-passes-unused-input
    rounds: [2]
    status: resolved
  - key: unpaused-run-lock-refusal-retry-never-succeeds
    rounds: [2]
    status: resolved
  - key: paused-hub-resume-form-no-input
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: busyerror-bare-teardown-literal
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised eleven findings, all resolved by round 2.
Round 2 raised seven; four are resolved in round 3 and three recur.
The BLOCKING recurrence is card 17: round 2's missing tests are now covered, but the `claudeengine` corpus replay of `master-fork-running` still pins the old shell-wait rule and the card names neither it nor its case file.
In card 11, a gone code branch beside another stopping branch still gives a refusal that names `lyx fabric remove`, because the entry's way forward is the first stopping branch's.
Card 2's `TestEnd` still lacks rows for the group check of a descendant that leads its own group.
Round 3 adds two keys: card 10's hub-pause resume form, which no input selects and no test reaches, and `busyError`'s read of owned processes meeting a bare `Teardown` literal in an integration test.
