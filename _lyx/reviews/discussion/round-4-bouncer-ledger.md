---
round: 4
ledger:
  - key: continue-pass-skips-owner-reconcile
    rounds: [1, 2]
    status: resolved
  - key: pair-teardown-claim-clear-restarts-removed-task
    rounds: [1]
    status: resolved
  - key: innerrun-memory-inventory-deferred
    rounds: [1, 2]
    status: resolved
  - key: pass-in-flight-through-inprocess-waits
    rounds: [1, 2]
    status: resolved
  - key: reed-down-ends-beam-dobby
    rounds: [1]
    status: resolved
  - key: continue-pass-flag-unbounded
    rounds: [1]
    status: resolved
  - key: halted-run-resumer-disagrees
    rounds: [1]
    status: resolved
  - key: respawn-cap-spent-in-harmless-gaps
    rounds: [1]
    status: resolved
  - key: third-live-batten-reader
    rounds: [1]
    status: resolved
  - key: darn-arbiter-copy-hub-layer-only
    rounds: [1]
    status: resolved
  - key: landing-count-misses-hand-passes
    rounds: [1]
    status: resolved
  - key: older-binaries-write-statusless-tasks
    rounds: [1]
    status: resolved
  - key: reexec-versus-first-pass
    rounds: [1]
    status: resolved
  - key: two-dedup-keys-for-dobby-notices
    rounds: [1]
    status: resolved
  - key: backoff-with-only-halted-runs
    rounds: [1]
    status: resolved
  - key: late-fswatch-event-after-pass-exit
    rounds: [1]
    status: resolved
  - key: d11-rationale-by-rejection
    rounds: [1]
    status: resolved
  - key: full-pass-takeover-loops-as-failing-pass
    rounds: [2]
    status: resolved
  - key: spawn-hold-yields-before-owner-recheck
    rounds: [2, 3]
    status: resolved
  - key: pass-one-check-unnamed
    rounds: [2]
    status: resolved
  - key: unreachable-origin-clears-owed-full-pass
    rounds: [2, 3]
    status: resolved
  - key: adopted-member-of-foreign-beam-has-no-dobby
    rounds: [2, 3]
    status: resolved
  - key: note-upsert-to-task-leaves-statusless
    rounds: [2, 3, 4]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: cleanup-removes-landing-before-count
    rounds: [2]
    status: resolved
  - key: failed-owner-pause-retried-on-stale-verdict
    rounds: [3, 4]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: draft-release-command-fails-as-written
    rounds: [3]
    status: resolved
  - key: owned-elsewhere-exit-code-defined-by-sites
    rounds: [4]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: adopt-refusal-follows-mutations
    rounds: [4]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 4 retired the round-3 BLOCKING spawn-hold key and the owed-full-pass, foreign-beam and draft-release keys: none recurs, and the review's settled-questions section confirms the release command works as typed.
The draft rule stays open on its commit point (F1, MEDIUM, now class decision): D5 must name `boardCriticalSection` as the place.
The owner-pause key stays open at LOW (F4), narrowed to the window before the first pause.
Two keys are new and LOW: the owned-elsewhere exit code defined by sites (F2) and the adopt refusal that can follow mutations (F3).
