---
round: 3
ledger:
  - key: failed-write-ownership-unbounded
    rounds: [1]
    status: resolved
  - key: clear-removes-pause-request-unbounded
    rounds: [1]
    status: resolved
  - key: pid-liveness-and-spawn-exclusivity
    rounds: [1]
    status: resolved
  - key: unwatched-envelope-staleness
    rounds: [1]
    status: resolved
  - key: loop-key-set-disagreement
    rounds: [1]
    status: resolved
  - key: teardown-kills-loop-after-reed-session
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
  - key: failure-stop-invariant-domain-too-wide
    rounds: [1]
    status: resolved
  - key: condition-lifetime-never-fires
    rounds: [1]
    status: resolved
  - key: second-failure-rule-ambiguous
    rounds: [1]
    status: resolved
  - key: pattern-cap-entry-misfiled
    rounds: [1]
    status: resolved
  - key: d10-restates-d4-stderr
    rounds: [1]
    status: resolved
  - key: loop-dies-without-envelope
    rounds: [2]
    status: resolved
  - key: condition-overwrite-unbounded
    rounds: [2]
    status: resolved
  - key: condition-ownership-class-mislabel
    rounds: [2]
    status: resolved
  - key: interrupted-error-omits-run-id
    rounds: [2]
    status: resolved
  - key: empty-key-drop-scope-disagreement
    rounds: [2]
    status: resolved
  - key: envelope-path-no-full-record
    rounds: [2]
    status: resolved
  - key: windows-start-time-gate-never-matches
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
  - key: windows-detach-stays-in-job-object
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: envelope-currency-blind-to-resume
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: dead-loop-stop-busy-pid-file-fate
    rounds: [3]
    status: open
    class: decision
    severity: MEDIUM
  - key: lock-race-loser-matches-loop-exited
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

The teardown key has now been open three rounds.
Round 1 and 2 found the session-end kill leaving the in-flight child running; round 3 finds that on Linux the parent-death signal reaches only the direct child, so the step's descendants outlive both the loop and the teardown.
Round 3 also found that the start-time gate cannot match on Windows, so the teardown kill never acts there.
The round-2 condition, ownership-class, empty-key and `run_id` items were not re-raised and the review's focus verdicts record them as settled.
New in round 3: the Windows job-object survival of the detached loop, an envelope currency key a resume does not change, the pid file's fate on a busy dead-loop stop, and a lock-race loser matching the loop-exited trigger.
