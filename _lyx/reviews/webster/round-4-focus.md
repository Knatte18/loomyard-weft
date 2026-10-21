---
round: 4
focus:
  - "internal/reedcli/watchdog_integration_test.go, the shared-loop setup of TestWatchdogDaemon: does the loop's idle window (`timing.IdleCycles`) comfortably exceed the longest down-then-up gap `DownThenUpDoesNotKillDaemon` can see under a loaded package run, and does any subtest of that loop depend on the loop idle-exiting?"
  - "PATTERN.md against `git show main:PATTERN.md`: apart from `PATTERN-origin-reconcile` and `PATTERN-push-failure-visible`, which this task declares, is every overview entry's text, `(test)` marker and `[background]` link identical to main, and is the overview still within its headroom line?"
  - "internal/fabricengine/destroy.go `restoreReplayTip` and `resetBack` in `cherryPickOnto`, and the `replayBolt` call site in replay.go: when `ResetKeep(startTip)` is refused after the branch moved, is the starting tip held by a tag through the destruction chokepoint, and does the returned error name it?"
  - "internal/fabricengine/destroy.go `cherryPickOnto`: does every error return after the reset, including the final `repo.CurrentSHA()` read, restore the starting tip via `resetBack`?"
---

Focus targets the open keys and the two new design findings from the judged round.
