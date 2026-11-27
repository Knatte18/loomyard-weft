---
round: 1
ledger:
    - key: refused-record-survives-owned-elsewhere-settle
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: failed-reset-or-handover-lost
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: design-wrong
    - key: beam-parent-pusher-never-restarted
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: test-only-fabric-methods
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: protected-branch-way-forward-missing
      rounds:
        - 1
      status: open
      class: scope
      severity: MEDIUM
      fix: mechanical
    - key: rebased-away-landing-commit-pushed
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: finalize-resume-releases-held-records
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: hub-lift-pushes-no-held-pair
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: webster-reset-push-no-owed-record
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: diverged-prime-pulls-as-level
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: parent-pusher-test-rows-missing
      rounds:
        - 1
      status: open
      class: scope
      severity: MEDIUM
      fix: mechanical
    - key: acceptance-behaviours-untested
      rounds:
        - 1
      status: open
      class: scope
      severity: MEDIUM
      fix: mechanical
    - key: further-push-coverage-gaps
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: beam-divergence-names-no-landing
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: unreachable-teardown-push-sentinels
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: reflog-passed-as-arguments
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: stale-push-descriptions
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: lym-cache-list-incomplete
      rounds:
        - 1
      status: open
      class: consistency
      severity: NIT
      fix: mechanical
---
Round 1 opens every key.
The gating defects cluster in the parent pusher (`internal/runpusher/parent.go`):
a refused record left standing, a reset or hand-over dropped on failure, and a removed commit still pushed.
The others are in paths that must start or hold a push after the pusher change:
beam parent restarts, the hub lift, `webster reset`, a Finalize resume and the diverged-prime pull.
The test gaps include untested paths of the parent pusher.
