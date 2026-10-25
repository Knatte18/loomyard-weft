---
round: 3
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pass-one-check-unnamed
    rounds: [2]
    status: resolved
  - key: unreachable-origin-clears-owed-full-pass
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: adopted-member-of-foreign-beam-has-no-dobby
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: note-upsert-to-task-leaves-statusless
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: cleanup-removes-landing-before-count
    rounds: [2]
    status: resolved
  - key: failed-owner-pause-retried-on-stale-verdict
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: draft-release-command-fails-as-written
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 3 raised six design findings.
Four sit on round-2 keys still open: the pause-covered pass refused before InnerRun (F1, BLOCKING, under the spawn-hold key), the dobby-only owed full pass (F2), the adopt refusal that omits a beam (F4), and the draft-giving writes that are not a closed set (F6).
Two keys are new: the failed owner pause retried on a stale verdict (F3), and the draft release command that fails as typed (F5).
Round 3 rejected its focus keys on owner-and-retry, the continue pass and the one-check-per-pass inventory, so the round-2 keys on those points are resolved.
