---
round: 2
ledger:
  - key: owed-push-still-writes-holding-record
    rounds: [1]
    status: resolved
  - key: prime-pusher-reads-no-batten-takeover
    rounds: [1]
    status: resolved
  - key: paused-pair-pusher-keeps-pushing
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 raised fifteen findings; round 2 resolved fourteen of them.
The one that survives is `paused-pair-pusher-keeps-pushing` (round 2's F1, carrying round 1's F3).
D7 names the status stop condition of `lym loom pause`, while D3's gate reads the hub pause record that the tender pause writes.
The fix names the tender pause and reads it through `pauserecord.ReadCoverage`.

Round 2 adds `code-branch-no-restart-after-records-push` (F2).
D6 leaves `CodeTipMissing` unset on the owner machine, which switches off the owner dobby's `settleCodeTip` push, and D5's dobby tick looks only at the records branch.
A pusher killed between its records push and its code push leaves the code tip local with no restart.
The fix either counts an ahead code branch in D5's dobby tick or names how `settleCodeTip` still pushes the held tip.
