---
round: 2
ledger:
  - key: pre-run-exits-silent-in-dobby-log
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: concurrent-adopt-checks-share-record
    rounds: [1]
    status: resolved
  - key: adopt-check-before-migration
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: reused-pid-keeps-dead-check-running
    rounds: [1]
    status: resolved
  - key: reexec-test-by-pid-alone
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: tick-and-owner-read-lines-leave-dobby-log
    rounds: [1]
    status: resolved
  - key: rotation-has-two-writers
    rounds: [1, 2]
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
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: worktree-gone-exit-line-has-no-destination
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fatal-run-error-after-reexec-not-posted
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: adopt-check-child-fds-unstated
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: foreign-landings-already-in-landed-since-tidy
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: worktree-gone-exit-leaves-tidy-in-flight
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: refused-reexec-false-alarm
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: adopt-envelope-names-either-check
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: heartbeat-bound-is-eleven-minutes
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's fifteen keys: nine no longer appear in round 2 and are resolved.
Four carry on: `pre-run-exits-silent-in-dobby-log` and `reexec-test-by-pid-alone` as MEDIUM design findings (a `placeDobby` refusal after a resolved worktree still leaves no trace; the start-ticks re-exec test does not reach `recordStart` or every pid writer), `adopt-check-before-migration` as a LOW design finding on the hand-run spawn-failure path, and `rotation-has-two-writers` as a LOW consistency finding on the spawn-time `dobby.out` rotation.
`concurrent-adopt-checks-share-record` is resolved; its remaining gap, the check child's own stdout and stderr, is filed as the new `adopt-check-child-fds-unstated`.
Round 2 added nine keys: four MEDIUM design (inherited fds, worktree-gone exit destination, fatal `Run` error after a re-exec, adopt check child fds), two LOW design, one LOW and two NIT consistency.
