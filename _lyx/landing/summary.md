# Reed: trace-id constant, quiet allow-set-title, watchdog stop fallback

Three small reed fixes.

## Trace id named through `logger.TraceIDEnv` in tests

`internal/reedengine/lifecycle_test.go`, `internal/shuttleengines/shuttleengines_test.go` and `internal/testkit/tmuxkit/tmuxkit_integration_test.go` now name the trace id variable as `logger.TraceIDEnv` instead of the `"LYX_TRACE_ID"` literal, so a rename of the variable cannot leave them testing a name lyx no longer sets.

## allow-set-title skipped on a tmux known to predate it

A strand launch on tmux 3.4 used to run `set-option -p allow-set-title off`, fail, and Warn once per pane.

- `toolfloor` declares `TmuxAllowSetTitle = "3.5"` and `TmuxHasAllowSetTitle`, which reads a `tmux -V` line and reports whether it parsed and whether it is at or above that release.
- `reedengine`'s per-path floor cache now stores a `tmuxFloorAnswer` holding the floor error and whether the parsed version lacks the option, from the same single probe.
  It logs the fallback at Info once, when it caches a parsed version below 3.5.
- `launchStrandLocked` skips the `set-option` when `TmuxCmd.lacksAllowSetTitle` says so.
  An unrecognised version line, Windows, and a hook-driven test command with no parseable floor line still try the option, and a failure still Warns and the launch continues.

The `toolfloor` and `reedengine` package docs describe the new behavior.

## Watchdog reads its stop marker on the timer when unwatched

The watchdog consumed the discover signal file only on a filesystem event.
When the watch failed to open, as with a full `fs.inotify.max_user_instances` table, it never read a stop, and `TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping`'s stop rows failed.

- `runWatchdogLoop` keeps an `unwatched` flag, set when the open fails or the event channel closes.
  While it is set, each timer wake first calls `consumeDiscoverSignal` and feeds a consumed signal through `discoveryWake.onSignal`, so a stop marker makes that cycle stop-woken.
  It adds no wakeup and keeps the backoff, so a stop without a watch lands within the discovery ceiling.
- The open failure's Warn names the fallback, and a closed channel Warns once the same way.
- `reedCLI` gains a `watchdogOpenWatch` seam, and the exit-path table gains a stop row driven through a failing opener.
  The existing stop rows now pass whether or not the real watch opens.
- `TestWatchdogDaemon`'s shared discovery loop, which must never see the stop a `down` writes before the next `up`, now gets an opener whose channel never delivers instead of one that fails, since a failing opener would now read that stop.
- The `reedcli` package doc and the loop's doc comment describe the fallback, and the `watchdogHubIdleCycles` comment no longer claims an unwatched daemon never sees a stop.

## Design decisions

- `tmuxkit.ServerEnviron` keeps its `"LYX_TRACE_ID"` literal, departing from the decision record, which planned to change it too.
  `tmuxkit` cannot import `logger`: `dotgit`'s in-package `TestMain` imports `tmuxkit`, and `logger` imports `dotgit`, so the import would form a cycle in `dotgit`'s test build.
  `TestEnvScrub_MatchesTmuxkitServerEnviron` now builds its expectation from `logger.TraceIDEnv`, so a rename that leaves the kit's literal behind fails that test.
- The fallback note is logged once per resolved tmux path per process, from the floor cache that is already per path, which in practice is once per process.
