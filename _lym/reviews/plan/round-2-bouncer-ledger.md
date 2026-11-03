---
round: 2
ledger:
  - key: card10-tier0-spawns-git
    rounds: [1]
    status: resolved
  - key: adopt-only-records-checkout-refusal-unreached
    rounds: [1]
    status: resolved
  - key: reused-records-worktree-blocks-ff-fetch
    rounds: [1]
    status: resolved
  - key: archive-compares-tips-before-fetch
    rounds: [1]
    status: resolved
  - key: sweeps-lock-after-state-read
    rounds: [1]
    status: resolved
  - key: batten-resume-refusal-lacks-follow-up
    rounds: [1]
    status: resolved
  - key: retry-widened-beyond-archive-tag-push
    rounds: [1]
    status: resolved
  - key: remove-warning-not-reaching-envelope
    rounds: [1]
    status: resolved
  - key: gone-pair-refusal-text-falsified
    rounds: [1]
    status: resolved
  - key: landing-push-noop-unbounded
    rounds: [1]
    status: resolved
  - key: push-anchored-no-production-caller
    rounds: [1]
    status: resolved
  - key: config-lock-row-cannot-fail
    rounds: [1]
    status: resolved
  - key: falsified-doc-text-outside-card-symbols
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: archive-test-reads-unexported-field
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d3-follow-up-sites-enumerated-by-hand
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: cleanup-live-pair-reread-unreached
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: start-first-pass-test-reads-own-fixture
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: task-worktree-gone-row-worded-backwards
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: follow-up-wording-asserted-three-times
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card10-integration-rows-repeat-add-coverage
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card4-warp-drop-widening-unstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card5-test-needlessly-serial
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: fabric-push-skip-has-no-test
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised 13 keys, one of them BLOCKING; round 2 re-read the fixes and found none regressed, so all but one resolved.
The exception is `falsified-doc-text-outside-card-symbols`, whose gap recurs in cards 8 and 10 as F6 (LOW).

Round 2 raised 10 further keys, none BLOCKING.
The one gating finding at MEDIUM is `d3-follow-up-sites-enumerated-by-hand` (F2), first raised this round.
The other two MEDIUM keys are consistency: `archive-test-reads-unexported-field` (F1) and `cleanup-live-pair-reread-unreached` (F3).
The remaining keys are LOW or NIT local corrections.
