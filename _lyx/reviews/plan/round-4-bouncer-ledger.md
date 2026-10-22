---
round: 4
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
    status: resolved
  - key: card17-stale-shell-wait-comments
    rounds: [2]
    status: resolved
  - key: gone-branch-remove-refusal-loops
    rounds: [2, 3]
    status: resolved
  - key: live-parent-branch-refusal-unreachable
    rounds: [2]
    status: resolved
  - key: end-identity-check-rows-missing
    rounds: [2, 3]
    status: resolved
  - key: run-lock-holder-passes-unused-input
    rounds: [2]
    status: resolved
  - key: unpaused-run-lock-refusal-retry-never-succeeds
    rounds: [2]
    status: resolved
  - key: paused-hub-resume-form-no-input
    rounds: [3]
    status: resolved
  - key: busyerror-bare-teardown-literal
    rounds: [3]
    status: resolved
  - key: await-run-lock-or-pass-refusals-unreached
    rounds: [4]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: reed-down-retry-step-mismatch
    rounds: [4]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: remove-verdict-negative-clause-unsatisfiable
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: claudeengine-doc-corpus-readings
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Rounds 1 to 3 raised twenty findings in total; every key from them is resolved by round 4.
Round 4's review settles card 2's group rows, card 10's `busyError` substrate, card 11's `report.Branches` source and card 17's replay coverage.
Round 4 raises four new keys, none design class: card 10 lacks a test that reaches `awaitRunLockOrPass`; card 12 wraps a refusal whose text already names the removal; card 11's `TestRemoveVerdict` negative clause cannot hold; card 17 leaves the `claudeengine` package doc at three readings.
