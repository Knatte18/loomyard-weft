---
round: 2
ledger:
  - key: beam-run-hold-blocks-membership
    rounds: [1]
    status: resolved
  - key: pawl-runs-fill-dobby-max-runs
    rounds: [1]
    status: resolved
  - key: landing-hold-defined-three-ways
    rounds: [1]
    status: resolved
  - key: fork-window-after-land-recheck
    rounds: [1]
    status: resolved
  - key: adopt-root-before-beam-strands-run
    rounds: [1]
    status: resolved
  - key: adopt-starts-beam-parts-mid-landing
    rounds: [1]
    status: resolved
  - key: ways-forward-name-shortcut
    rounds: [1]
    status: resolved
  - key: ensurelive-callers-restart-beam-dobby
    rounds: [1]
    status: resolved
  - key: runproc-role-scoped-end-deferred
    rounds: [1]
    status: resolved
  - key: resume-owner-claim-left-in-place
    rounds: [1]
    status: resolved
  - key: watcher-owner-read-failure-direction
    rounds: [1]
    status: resolved
  - key: can-land-trigger-and-episode-memory
    rounds: [1]
    status: resolved
  - key: beam-await-crash-leaves-beam-stuck
    rounds: [1]
    status: resolved
  - key: no-root-owner-sentence-contradiction
    rounds: [1]
    status: resolved
  - key: d3-before-writes-contradicts-d1
    rounds: [1]
    status: resolved
  - key: entry-no-adopt-of-beam-unread
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-gaps
    rounds: [1]
    status: resolved
  - key: d11-carries-deliberation
    rounds: [1]
    status: resolved
  - key: beamlandrefusal-orch-blocker-pawl-callers
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: accept-retry-leaves-dobby-down
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: accept-stop-before-hold-write-window
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: accept-recheck-refuses-own-accept
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: beam-await-driver-respawn-cap-silent
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: run-lock-lift-unbounded
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: mixed-version-bound-false-for-recipe-pawl
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: adopt-root-alone-leaves-beam-undriven
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: cancel-refusal-reads-stale-bolt
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: anchor-lost-on-rerun-after-died-first-pass
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: finalize-backstop-names-refused-way-forward
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-ensurelive-caller-list-incomplete
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: criterion-tests-case-that-cannot-arise
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's eighteen keys are marked resolved: round 2's fresh review does not raise any of them again.
Round 2 raised thirteen new findings, one BLOCKING.
The blocker is that `BeamLandRefusal` keeps its live-orch blocker while the beam's orch runs until Beam-Down, so Finalize, cancel and the can-land notice all refuse.
The accept step's ordering is the second cluster: a retried accept leaves the dobby down, the dobby can restart before the hold is written, and the re-check refuses its own accept.
The rest are unbounded guards (run lock, mixed version, driver respawn cap, cancel on stale Bolt) and local corrections to D1, D6, D9, D10 and the acceptance criteria.
