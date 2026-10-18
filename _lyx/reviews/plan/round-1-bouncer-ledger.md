---
round: 1
ledger:
  - key: pattern-md-headroom-overrun
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: command-entry-refusals-no-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: child-in-board-breaks-on-subpath-anchor
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: empty-pause-slug-lands-under-board-entry
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: remove-respawn-pushes-prime-not-pair
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: spawn-pair-push-test-cannot-survive-card-7
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: origin-writes-local-marked-test-wrong-package
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: local-refusal-way-forward-fails-own-input
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: pending-flag-protocol-no-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: verify-skips-pattern-and-docs-tests
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card-12-verbs-no-bound
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: command-key-error-points-at-unread-layer
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: push-lock-busy-misreports-after-landed-push
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: local-only-gate-refusal-unreached-input
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: loom-park-test-missing-from-card-10
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: run-row-command-allowlist-reason-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: window-envelope-text-names-launcher
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 17 findings, all open.
Gating: the headroom overrun from cards 10 and 15 (F1), the missing way-forward clauses on `commandEntry` refusals (F2), the `_board` working-directory failure on subpath-anchored hubs (F3), the unbounded pending-flag protocol (F9), and the verify block skipping `internal/pattern` and `internal/lyxcwd` (F10).
The rest are consistency fixes on cards 4 to 10, 12, 14 and 15, and one scope gap (`TestLoomPark` on card 10).
