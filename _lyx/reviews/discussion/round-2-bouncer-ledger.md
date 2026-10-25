---
round: 2
ledger:
  - key: continue-pass-skips-owner-reconcile
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pair-teardown-claim-clear-restarts-removed-task
    rounds: [1]
    status: resolved
  - key: innerrun-memory-inventory-deferred
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pass-in-flight-through-inprocess-waits
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: spawn-hold-yields-before-owner-recheck
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pass-one-check-unnamed
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: unreachable-origin-clears-owed-full-pass
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: adopted-member-of-foreign-beam-has-no-dobby
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: note-upsert-to-task-leaves-statusless
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: cleanup-removes-landing-before-count
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 2 raised ten findings.
Of the seventeen round-1 keys, three are still open: the owner-reconcile skip on continue passes (F3, reopened as BLOCKING), the InnerRun memory inventory (F5, now the missing cross-pass counters), and the in-pass waits list (F9, now an incomplete list).
The other fourteen round-1 keys drew no round-2 finding and are resolved.
New in round 2: the full-pass takeover loop (F1) and the spawn hold yielding before its owner re-check (F2), both BLOCKING; the unnamed pass check (F4), unreachable-origin clearing of the owed pass (F6), the adopted member of a foreign beam (F7) and the statusless note upsert (F8), all MEDIUM; the cleanup-versus-count window (F10), LOW.
