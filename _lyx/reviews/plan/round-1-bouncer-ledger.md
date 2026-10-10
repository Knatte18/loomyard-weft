---
round: 1
ledger:
  - key: pattern-overview-over-cap
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: arming-refusal-reads-as-interrupted
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: loop-lock-liveness-race
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: shedverbs-trace-path-join
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: interrupted-stops-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: tests-drive-unexported-helpers
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: waiter-fswatch-misses-dead-loop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: loop-detached-flag-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: windows-code-not-compiled
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card19-decide-resign-callers
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: report-arm-error-test-overlap
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: sigquit-no-grace
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: stale-pid-file-after-envelope
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: kills-not-logged
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: findings-file-wrong-scratch-dir
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: windows-killtree-unnamed-job
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card8-bundles-three-units
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: failure-stop-invariant-early
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: start-go-header-stale
    rounds: [1]
    status: open
    class: scope
    severity: NIT
---

Round 1 raised nineteen findings, all still open.
The structural cluster is the loop and waiter design in cards 10 and 11: the lock-liveness race, the fswatch-blind dead loop, the unbounded hidden flag and the stale pid file.
The overview byte cap (F1) and the arming-refusal path (F2) are the other design blockers.
The `PATTERN` citations are the trace-path join in `shedverbs`, the missing way-forward text and the helper-driven tests.
