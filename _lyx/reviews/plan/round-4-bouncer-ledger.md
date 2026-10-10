---
round: 4
ledger:
  - key: pattern-overview-over-cap
    rounds: [1]
    status: resolved
  - key: arming-refusal-reads-as-interrupted
    rounds: [1, 2, 3]
    status: resolved
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
    rounds: [2, 3, 4]
    status: open
    class: design
    severity: LOW
  - key: arming-refusal-row-no-way-forward
    rounds: [2]
    status: resolved
  - key: session-end-respawn-window
    rounds: [2, 3, 4]
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
    status: resolved
  - key: detached-loop-arming-refusal-lost-to-log
    rounds: [3]
    status: resolved
---

Round 1 raised nineteen keys; round 2 reopened two in narrower form and added seven; round 3 resolved most and added two LOW keys.
Round 4 raised two findings, both `design`, none BLOCKING.

- `session-end-respawn-window` (F1, MEDIUM) survives a third round, narrower again: the teardown's pid-file mark bars only the first invocation after the session end, because card 12's `deadLoopStop` retires it and writes an envelope the next invocation delivers.
- `second-waiter-loses-envelope` (F2, LOW) survives a third round: a waiter that never read its own loop's pid file can adopt or misreport a later loop's records.
- `arming-refusal-reads-as-interrupted`, `arming-refusal-dead-loop-write-order` and `detached-loop-arming-refusal-lost-to-log` are resolved; round 4's focus answers 2 and 4 settled them as not defects.
