---
round: 2
ledger:
  - key: slug-pause-exempt-for-lock-holder
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: adopt-back-replays-old-owner-commits
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: shed-verb-check-vs-batten-child-spawns
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-halt-way-forward
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: offline-fallback-unbounded
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: parent-lock-outlives-handover
    rounds: [1]
    status: resolved
  - key: start-check-after-seed-claim-order
    rounds: [1]
    status: resolved
  - key: adopt-all-vs-done-and-unstarted
    rounds: [1]
    status: resolved
  - key: constraints-owner-reads-understated
    rounds: [1]
    status: resolved
  - key: shed-resume-unowned-run-held
    rounds: [1]
    status: resolved
  - key: owner-pause-marking-identity
    rounds: [1]
    status: resolved
  - key: owner-write-lost-replay-unchecked
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-gaps
    rounds: [1]
    status: resolved
  - key: d8-same-machine-resume-window
    rounds: [1]
    status: resolved
  - key: d7-deliberation-narrative
    rounds: [1]
    status: resolved
---

Round 2 reopens four round-1 keys and raises one new key.
F1 (BLOCKING) is `slug-pause-exempt-for-lock-holder`: round 1's fix made the lock release the thing that lets the slug pause hold a run, and the later carve-out for a run past its parent-side merge undoes that for one run.
F2 is `adopt-back-replays-old-owner-commits`: the step-1 stop misses a live pair.
F3 is `shed-verb-check-vs-batten-child-spawns`: the fix covered only the re-step spawn, but every InnerRun spawn of `lyx loom start --no-attach` on an existing run meets the check.
F4 is `d7-halt-way-forward`: the named adopt call stops on step 1 first.
F5 is the new `offline-fallback-unbounded`: the offline resume has no stated bound.
The ten other round-1 keys do not appear in round 2 and are resolved.
