---
round: 2
ledger:
  - key: pattern-md-headroom-overrun
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: command-entry-refusals-no-way-forward
    rounds: [1, 2]
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
    rounds: [1, 2]
    status: open
    class: decision
    severity: LOW
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
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: card-15-nilseams-edit-wrong
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: command-row-background-link-uncreated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: verify-misses-loomrecipe
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised 17 findings.
Round 2's fresh review re-raised only the PATTERN.md headroom (now as an unnamed cut in card 10), the pending-flag bound (now WorkDir and slugs) and the commandEntry refusal clauses (now one untested path); the other round 1 keys are resolved.
New in round 2: the declared `PATTERN-recipe-setting-lookup` line is one character over the cap (BLOCKING), card 15's `NilSeams` edit is wrong, card 10 counts a background link no card creates, and `internal/loomrecipe` is missing from `## verify:`.
