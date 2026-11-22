---
round: 3
ledger:
  - key: owed-push-still-writes-holding-record
    rounds: [1]
    status: resolved
  - key: prime-pusher-reads-no-batten-takeover
    rounds: [1]
    status: resolved
  - key: paused-pair-pusher-keeps-pushing
    rounds: [1, 2]
    status: resolved
  - key: pusher-runproc-owner-is-slug
    rounds: [1]
    status: resolved
  - key: handover-pair-removal-flag-open
    rounds: [1]
    status: resolved
  - key: driver-stop-record-pushes-not-in-scope
    rounds: [1]
    status: resolved
  - key: code-tip-window-reads-as-wait
    rounds: [1]
    status: resolved
  - key: refused-prime-push-stops-every-batten-run
    rounds: [1]
    status: resolved
  - key: teardown-unbounded-wait-under-prime-lock
    rounds: [1]
    status: resolved
  - key: d1-no-wait-vs-d5-teardown-wait
    rounds: [1]
    status: resolved
  - key: d4-restates-existing-behaviour
    rounds: [1]
    status: resolved
  - key: no-upstream-branch-no-dobby-restart
    rounds: [1]
    status: resolved
  - key: sleeping-pusher-lock-wording
    rounds: [1]
    status: resolved
  - key: d3-start-rule-omits-refused-record
    rounds: [1]
    status: resolved
  - key: spawnstatuspush-not-unused
    rounds: [1]
    status: resolved
  - key: code-branch-no-restart-after-records-push
    rounds: [2]
    status: resolved
---

Round 1 raised fifteen findings, and round 2 resolved fourteen.
Round 3 resolved the last two.
`paused-pair-pusher-keeps-pushing` closed once D7 named `tendercli.PausePair` and read the pause through `pauserecord.ReadCoverage`.
`code-branch-no-restart-after-records-push` closed once D5's tick counted a pair whose local code branch holds the code tip named by the records tip's `Warp-SHA` and whose origin ref lacks it, with D6 naming the tick as the restart.
Round 3 raised no findings.
