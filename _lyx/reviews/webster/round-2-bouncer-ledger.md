---
round: 2
ledger:
  - key: replay-tags-after-reset
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: unclassified-push-failure-unrecorded
    rounds: [1]
    status: resolved
  - key: reconcile-verdict-no-slug-filter
    rounds: [1]
    status: resolved
  - key: pattern-entries-rewritten-undeclared
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: set-aside-paths-in-dirtiness-check
    rounds: [1]
    status: resolved
  - key: resetkeep-failure-reads-dirty
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: retry-failure-hides-cause
    rounds: [1]
    status: resolved
  - key: weft-error-stalls-warp-arm
    rounds: [1]
    status: resolved
  - key: watchdog-down-up-idle-window
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: settle-unmoved-weft-swallows-read-error
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1 opened eight findings; round 2 resolves five of them and keeps three open.
The set-aside tagging finding returned as BLOCKING: tagging after the branch moves still leaves commits reachable only through the reflog when the restore fails or `replayBolt` errors after the move.
The `ResetKeep` error mapping returned at MEDIUM, and the undeclared PATTERN overview rewrites remain a MEDIUM decision for `ly:orch`.
Two keys are new: the Publish verify failure of `TestWatchdogDaemon/DownThenUpDoesNotKillDaemon`, whose idle window is shorter than a down-then-up under load, and `settleUnmovedWeft` turning a failed local read into a success with `weft_pulled` false.
The push-failure recording and the reconcile verdict's slug filter were re-examined and rejected as defects.
