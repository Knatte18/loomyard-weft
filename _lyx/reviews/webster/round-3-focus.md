---
round: 3
focus:
  - "internal/fabricengine/replay.go, `replayWeft` and `replayBolt`: after any failure, is every commit the replay set aside reachable from a tag, and does the branch end at its starting tip whenever the replay did not complete, with no commit named only by error text or the reflog? Check each error return after `cherryPickOnto` and each test row that pins a tagging or by-entry failure."
  - "internal/fabricengine/destroy.go, `cherryPickOnto`: is `ErrReplayDirty` wrapped only around git's local-change refusal of the reset, and does a reset failure of another kind (for example a held index lock) surface as a different error naming the git failure?"
  - "internal/fabricengine/pull.go, `settleUnmovedWeft`: does a failed local read of the weft's upstream, head or ahead and behind sets return an error, while only the missing-upstream and nothing-behind cases warn and continue, so `Pull` never returns success with `weft_pulled` false on an infrastructure error?"
  - "internal/reedcli/watchdog_integration_test.go, `TestWatchdogDaemon/DownThenUpDoesNotKillDaemon`: does the daemon's idle window outlast the test's own down-then-up gap under a loaded package run, and are the other watchdog tests left on their existing timing?"
  - "PATTERN.md and pattern/: does every overview entry other than `PATTERN-origin-reconcile` and `PATTERN-push-failure-visible` read as on main, or is each differing entry declared among the task's PATTERN changes?"
---

Round 3 checks the five sites above against the current diff.
