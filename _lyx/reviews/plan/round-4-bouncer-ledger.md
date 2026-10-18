---
round: 4
ledger:
  - key: pattern-md-headroom-overrun
    rounds: [1, 2, 3]
    status: resolved
  - key: command-entry-refusals-no-way-forward
    rounds: [1, 2, 3]
    status: resolved
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
    rounds: [1, 2, 3, 4]
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
    status: resolved
  - key: card-15-testcommandentry-fixture-seams
    rounds: [4]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised 17 findings, round 2 seven, round 3 four, round 4 two; round 4 has no BLOCKING finding.
The `PATTERN.md` headroom overrun and the two card-15 test-fixture gaps from round 3 are not re-raised and are resolved.
Still open: the pending-flag respawn bound in card 8, now in its fourth round, because each round's rows covered one more side or field and left another unreached; the fix asks for a side-by-field table.
New in round 4: card 15's `TestCommandEntry` fixtures name Env seams piecemeal and leave `Command.LogPath` and `Command.Dir` unset, so the timeout rows would reach nil funcs.
