# Reed small fixes: stop marker without inotify, quiet allow-set-title

Three small reed fixes.

## What changes

### Watchdog reads its stop marker without a watch

The hub watchdog consumed the discover signal file, which carries the stop marker, only through its filesystem watch.
When the watch failed to open, as with a full `fs.inotify.max_user_instances` table, `events` was nil and a stop was never read.
`TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping` failed for the same reason.

`runWatchdogLoop` in `internal/reedcli` now keeps an `unwatched` flag, set when the open fails or the event channel closes.
While it is set, each timer wake first calls `consumeDiscoverSignal` and feeds a consumed signal to `discoveryWake.onSignal`, so a stop marker makes that cycle stop-woken.
The open failure and a closed channel each log once at Warn, naming the fallback.
A consume error on the timer arm goes through `logger.Episode`, so a persistent fault logs once per episode and not once per wake.

`watchdogCmd` gets a watch-opener seam.
`TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping` gains stop rows driven through a failing opener and a closed channel, and asserts the fallback trace lines.

### No allow-set-title Warn per pane on tmux below 3.5

`internal/toolfloor` declares `TmuxAllowSetTitle = "3.5"` and a function reporting whether a `tmux -V` line parsed and is at or above it.
The `reedengine` floor cache stores that answer per resolved tmux path, from the same single probe, and logs the fallback once per path per process at Info.
`launchStrandLocked` skips the `allow-set-title` `set-option` when the parsed version is below 3.5.
Title drift stays reported by `lyx reed list`.

### Trace id named through the constant

`lifecycle_test.go`, `shuttleengines_test.go` and `tmuxkit_integration_test.go` use `logger.TraceIDEnv` instead of the `"LYX_TRACE_ID"` literal, so a rename cannot leave them testing a name lyx no longer sets.

Closes #783

## Design decisions

- An unparsed version line, a hook-driven command with no floor line, and Windows count as having the option.
  The `set-option` still runs there and its failure still Warns, so only a positively identified old tmux is silenced.
- Skipping the option on a parsed version below 3.5 only lets an agent overwrite the pane title, which the launch already tolerated after the failed call.
- The unwatched fallback adds no wakeup and keeps the backoff, so `PATTERN-daemon-wakeups` holds.
  Stop latency without a watch is bounded by the discovery ceiling.
- `TestWatchdogDaemon`'s shared discovery loop uses an opener returning a channel that never delivers, not a failing one.
  With the fallback, a failing opener would let that loop read the stop that `Down` writes and exit before `Up`, the race the test excludes on purpose.
- `tmuxkit.ServerEnviron` keeps its `"LYX_TRACE_ID"` literal, since `logger` reaches `dotgit`, whose in-package `TestMain` imports `tmuxkit`, so using `logger.TraceIDEnv` there would form an import cycle.
  `TestEnvScrub_MatchesTmuxkitServerEnviron` fails on a rename that leaves the literal behind.
- `internal/toolfloor` stays a stdlib-only leaf.
- Docs: `internal/reedengine/doc.go`, `internal/reedcli/doc.go` and `internal/toolfloor/doc.go` describe the new behavior.

## To check by hand

- `internal/testkit/tmuxkit/tmuxkit_integration_test.go` is tmux-tier and was not run; its change is a constant substitution checked only by compiling.
- On a tmux 3.4 host such as Ubuntu 24.04, `lyx orch start --continue` should print no `allow-set-title` Warn per pane.
