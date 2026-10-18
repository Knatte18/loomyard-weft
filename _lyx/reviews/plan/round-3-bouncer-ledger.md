---
round: 3
ledger:
  - key: pattern-md-headroom-overrun
    rounds: [1, 2, 3]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: command-entry-refusals-no-way-forward
    rounds: [1, 2, 3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: child-in-board-breaks-on-subpath-anchor
    rounds: [1]
    status: resolved
  - key: empty-pause-slug-lands-under-board-entry
    rounds: [1]
    status: resolved
  - key: remove-respawn-pushes-prime-not-pair
    rounds: [1]
    status: resolved
  - key: spawn-pair-push-test-cannot-survive-card-7
    rounds: [1]
    status: resolved
  - key: origin-writes-local-marked-test-wrong-package
    rounds: [1]
    status: resolved
  - key: local-refusal-way-forward-fails-own-input
    rounds: [1]
    status: resolved
  - key: pending-flag-protocol-no-bound
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: verify-skips-pattern-and-docs-tests
    rounds: [1]
    status: resolved
  - key: card-12-verbs-no-bound
    rounds: [1]
    status: resolved
  - key: command-key-error-points-at-unread-layer
    rounds: [1]
    status: resolved
  - key: push-lock-busy-misreports-after-landed-push
    rounds: [1]
    status: resolved
  - key: local-only-gate-refusal-unreached-input
    rounds: [1]
    status: resolved
  - key: loom-park-test-missing-from-card-10
    rounds: [1]
    status: resolved
  - key: run-row-command-allowlist-reason-stale
    rounds: [1]
    status: resolved
  - key: window-envelope-text-names-launcher
    rounds: [1]
    status: resolved
  - key: declared-recipe-setting-line-over-cap
    rounds: [2]
    status: resolved
  - key: card-15-nilseams-edit-wrong
    rounds: [2]
    status: resolved
  - key: command-row-background-link-uncreated
    rounds: [2]
    status: resolved
  - key: verify-misses-loomrecipe
    rounds: [2]
    status: resolved
  - key: command-minimalentries-env-setting-nil
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised 17 findings, round 2 seven, round 3 four.
Still open: the `PATTERN.md` headroom overrun (BLOCKING, now 211 bytes owed with card 10 stopping to ask `ly:orch`), the pending-flag respawn bound (slugs and, new in round 3, the opposite-side path), the missing `command` row in `TestCommandEntry`, and the new `Env.Setting` nil gap in card 15's test fixtures.
Round 2's setting-line cap conflict, `NilSeams` edit, background link and `loomrecipe` verify findings are not re-raised.
