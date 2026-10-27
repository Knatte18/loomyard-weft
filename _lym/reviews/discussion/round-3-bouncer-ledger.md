---
round: 3
ledger:
  - key: pre-run-exits-silent-in-dobby-log
    rounds: [1, 2]
    status: resolved
  - key: concurrent-adopt-checks-share-record
    rounds: [1]
    status: resolved
  - key: adopt-check-before-migration
    rounds: [1, 2]
    status: resolved
  - key: reused-pid-keeps-dead-check-running
    rounds: [1]
    status: resolved
  - key: reexec-test-by-pid-alone
    rounds: [1, 2]
    status: resolved
  - key: tick-and-owner-read-lines-leave-dobby-log
    rounds: [1]
    status: resolved
  - key: rotation-has-two-writers
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: unreadable-machine-name-unbounded
    rounds: [1]
    status: resolved
  - key: fresh-dobby-unregistered-window
    rounds: [1]
    status: resolved
  - key: two-behaviours-lack-criterion
    rounds: [1]
    status: resolved
  - key: scope-out-bullet-wrong-entry
    rounds: [1]
    status: resolved
  - key: decision-4-reason-mismatch
    rounds: [1]
    status: resolved
  - key: migration-not-once
    rounds: [1]
    status: resolved
  - key: decision-8-bound-lacks-evidence
    rounds: [1]
    status: resolved
  - key: constraints-bullet-repeats
    rounds: [1]
    status: resolved
  - key: inherited-fds-keep-raw-output-on-dobby-log
    rounds: [2]
    status: resolved
  - key: worktree-gone-exit-line-has-no-destination
    rounds: [2]
    status: resolved
  - key: fatal-run-error-after-reexec-not-posted
    rounds: [2]
    status: resolved
  - key: adopt-check-child-fds-unstated
    rounds: [2]
    status: resolved
  - key: foreign-landings-already-in-landed-since-tidy
    rounds: [2]
    status: resolved
  - key: worktree-gone-exit-leaves-tidy-in-flight
    rounds: [2]
    status: resolved
  - key: refused-reexec-false-alarm
    rounds: [2]
    status: resolved
  - key: adopt-envelope-names-either-check
    rounds: [2]
    status: resolved
  - key: heartbeat-bound-is-eleven-minutes
    rounds: [2]
    status: resolved
  - key: worktree-gone-root-recreated-by-state-access
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: ticksless-reexec-notices-unstated
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Rounds 1 and 2 raised twenty-four keys; round 3 no longer carries any of them but `rotation-has-two-writers`, so the rest are resolved.
`rotation-has-two-writers` is in its third round as a LOW consistency finding: the only-the-lock-holder rule needs scoping to `dobby.log`, with the Windows `dobby.out` spawner named as the exception (F3).
Round 3 adds two keys.
F1 is a MEDIUM design finding: the dobby's own state access recreates a removed worktree root, so the root-gone exit can be defeated and its tidy stop reads a fresh `State`.
F2 is a LOW consistency finding: a ticks-less record's status as a re-exec for Decision 3's notices is unstated.
