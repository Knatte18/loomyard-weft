---
round: 3
ledger:
  - key: pattern-overview-over-cap
    rounds: [1]
    status: resolved
  - key: arming-refusal-reads-as-interrupted
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: loop-lock-liveness-race
    rounds: [1, 2]
    status: resolved
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
    rounds: [2, 3]
    status: open
    class: design
    severity: LOW
  - key: arming-refusal-row-no-way-forward
    rounds: [2]
    status: resolved
  - key: session-end-respawn-window
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: until-stop-inflight-record
    rounds: [2]
    status: resolved
  - key: failure-stop-pattern-line-shape
    rounds: [2]
    status: resolved
  - key: run-lock-wait-wrong-pid
    rounds: [2]
    status: resolved
  - key: allowlist-not-in-overview-entry
    rounds: [2]
    status: resolved
  - key: arming-refusal-dead-loop-write-order
    rounds: [3]
    status: open
    class: design
    severity: LOW
  - key: detached-loop-arming-refusal-lost-to-log
    rounds: [3]
    status: open
    class: design
    severity: LOW
---

Round 1 raised nineteen keys; round 2 reopened two of them in narrower form and added seven.
Round 3 resolved every round-2 key but three, which it reopened narrower again, and added two LOW keys.

- `arming-refusal-reads-as-interrupted` fell from BLOCKING to MEDIUM: the shed subtree is settled, the loom subtree's pre-run still has no location when `wire` fails (F2).
- `session-end-respawn-window` survives only for the teardown holding the loop lock with no pid file (F1).
- `second-waiter-loses-envelope` is now the loop-id comparison and the single delivered slot (F5), LOW.
- New: `arming-refusal-dead-loop-write-order` (F3) and `detached-loop-arming-refusal-lost-to-log` (F4), both LOW.
