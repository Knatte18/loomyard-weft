---
round: 1
ledger:
  - key: beam-run-hold-blocks-membership
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pawl-runs-fill-dobby-max-runs
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: landing-hold-defined-three-ways
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: fork-window-after-land-recheck
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: adopt-root-before-beam-strands-run
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: adopt-starts-beam-parts-mid-landing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: ways-forward-name-shortcut
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: ensurelive-callers-restart-beam-dobby
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: runproc-role-scoped-end-deferred
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: resume-owner-claim-left-in-place
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: watcher-owner-read-failure-direction
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: can-land-trigger-and-episode-memory
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: beam-await-crash-leaves-beam-stuck
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: no-root-owner-sentence-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-before-writes-contradicts-d1
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: entry-no-adopt-of-beam-unread
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: acceptance-criteria-gaps
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d11-carries-deliberation
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 18 findings against the decision record; none recurs, as this is the first round.
The gating core is the beam entry's run hold (F1, F3, F4, F6, F13) and the parent dobby's `max_runs` (F2): the pawl run's long stay at Beam-Await collides with readers that treat any run hold as in flight.
Adopt ordering and restarts (F5, F6, F8) are the second cluster.
The rest are local corrections to D3, D4, D5, D7, D11 and the acceptance criteria.
