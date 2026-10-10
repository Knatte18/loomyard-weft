---
round: 2
ledger:
  - key: pattern-overview-over-cap
    rounds: [1]
    status: resolved
  - key: arming-refusal-reads-as-interrupted
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: loop-lock-liveness-race
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: shedverbs-trace-path-join
    rounds: [1]
    status: resolved
  - key: interrupted-stops-no-way-forward
    rounds: [1]
    status: resolved
  - key: tests-drive-unexported-helpers
    rounds: [1]
    status: resolved
  - key: waiter-fswatch-misses-dead-loop
    rounds: [1]
    status: resolved
  - key: loop-detached-flag-unbounded
    rounds: [1]
    status: resolved
  - key: windows-code-not-compiled
    rounds: [1]
    status: resolved
  - key: card19-decide-resign-callers
    rounds: [1]
    status: resolved
  - key: report-arm-error-test-overlap
    rounds: [1]
    status: resolved
  - key: sigquit-no-grace
    rounds: [1]
    status: resolved
  - key: stale-pid-file-after-envelope
    rounds: [1]
    status: resolved
  - key: kills-not-logged
    rounds: [1]
    status: resolved
  - key: findings-file-wrong-scratch-dir
    rounds: [1]
    status: resolved
  - key: windows-killtree-unnamed-job
    rounds: [1]
    status: resolved
  - key: card8-bundles-three-units
    rounds: [1]
    status: resolved
  - key: failure-stop-invariant-early
    rounds: [1]
    status: resolved
  - key: start-go-header-stale
    rounds: [1]
    status: resolved
  - key: second-waiter-loses-envelope
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: arming-refusal-row-no-way-forward
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: session-end-respawn-window
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: until-stop-inflight-record
    rounds: [2]
    status: open
    class: scope
    severity: LOW
  - key: failure-stop-pattern-line-shape
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: run-lock-wait-wrong-pid
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: allowlist-not-in-overview-entry
    rounds: [2]
    status: open
    class: scope
    severity: LOW
---

Round 1's nineteen keys are resolved except two, which round 2 reopened in narrower form.
The arming refusal now fails at the waiter's own pre-run rather than in the child, so it still never reaches the loop and leaves the status `running` (BLOCKING).
The lock-liveness race now survives for an invocation that read a probe-held lock and spawned nothing.
Round 2 added seven keys: the second waiter losing the envelope, the bootstrap arming row's missing way forward, the session-end respawn window, the never-finished in-flight record, the pattern line shape, the wrong pid in the run-lock wait, and the allowlist missing from the overview entry.
