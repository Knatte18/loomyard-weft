# Decision record — reed-small-fixes

## Goal

Three small reed fixes: tests name the trace id variable through `logger.TraceIDEnv`, a strand launch on tmux below 3.5 stops warning about `allow-set-title` once per pane, and the hub watchdog reads its stop marker on its timer cycle when its filesystem watch is unavailable.

## Scope

- `internal/reedengine/lifecycle_test.go`, `internal/shuttleengines/shuttleengines_test.go` and `internal/testkit/tmuxkit/tmuxkit_integration_test.go` name the trace id variable as `logger.TraceIDEnv` instead of the `"LYX_TRACE_ID"` literal.
- `internal/testkit/tmuxkit/tmuxkit.go`'s `ServerEnviron` filter names it the same way.
- `internal/toolfloor` declares the tmux release that added `allow-set-title` and reports whether a `tmux -V` line is at or above it.
- `internal/reedengine` caches that answer beside the floor answer, logs the fallback once per tmux path per process at Info, and skips the `allow-set-title` `set-option` on a tmux known to lack it.
- `internal/reedcli`'s watchdog loop consumes the discover signal file on each timer cycle while no watch delivers its events, from a failed open or a closed event channel, and logs that once at Warn.
- `watchdogCmd` gets a file-watch opener test seam, and `TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping` gains a stop row driven through a failing opener.
- Docs: `internal/reedengine/doc.go`, `internal/toolfloor/doc.go` and the watchdog loop's doc comment describe the new behavior.

## Decisions

### allow-set-title gated on the version the floor probe already read

Decision: `toolfloor` gains `TmuxAllowSetTitle = "3.5"` and a function reporting, for a `tmux -V` line, whether it parsed and whether it is at or above that release.
`tmuxFloorCache` stores that answer per resolved path with the floor answer, from the same single probe, and logs at Info once, when it caches a parsed version below 3.5.
`launchStrandLocked` skips the `set-option` when the answer says the option is missing.
An unparsed line, a hook-driven command with no floor line, and Windows count as having the option, so the `set-option` runs and its failure still Warns.

Rationale: the floor probe already runs `tmux -V` once per path per process, so the gate costs no extra subprocess, and the fallback note lands once per process instead of once per pane.
The guard it bounds is display-only: skipping the `set-option` lets the agent overwrite a pane title, which the launch already tolerated after the failed call, and it applies only to a parsed version below 3.5.

### Watchdog reads the stop marker on its timer cycle when unwatched

Decision: the loop keeps a flag set when the watch open fails or its event channel closes; while it is set, each timer wake first calls `consumeDiscoverSignal` and feeds a consumed signal through `discoveryWake.onSignal`, so a stop marker makes that cycle stop-woken.
The open failure's Warn names the fallback, and a closed channel Warns once the same way.

Rationale: the stop then works with a full inotify table, at the backed-off cadence, with no extra wakeup, so PATTERN-daemon-wakeups holds.
The stop latency without a watch is bounded by the discovery ceiling, which the idle-cycle exit already exceeded.

### The shared integration loop keeps seeing no stop

Decision: `TestWatchdogDaemon`'s shared discovery loop, which relies on never seeing a stop across a down followed by an up, gets an opener returning a channel that never delivers instead of one that fails.

Rationale: with the fallback, a failing opener would let that loop read the stop `Down` writes and exit in the gap before `Up`, which is the race the test excludes on purpose.

### Trace id named through the constant, test kit included

Decision: the three named test files and `tmuxkit.ServerEnviron` use `logger.TraceIDEnv`.

Rationale: `tmuxkit.ServerEnviron` is the filter the shuttleengines test pins against the engines' scrub, so a literal there would let a rename leave the kit filtering a name lyx no longer sets; `logger`'s tests import `tmuxkit` only from the external `logger_test` package, so no import cycle forms.

## Constraints

- `internal/toolfloor` stays a stdlib-only leaf (PATTERN-leaf-packages).
- The watchdog adds no wakeup and keeps its backoff (PATTERN-daemon-wakeups).
- Tests follow PATTERN-test-economy: new cases are rows in the tests that already cover the behavior.
- Go comments carry one sentence per line (PATTERN-comment-line-breaks).

## Auto-mode assumptions

- The board entry's "once per process" log is met by once per resolved tmux path per process, since a process uses one tmux path in practice.
- The tmuxkit non-test literal is the same defect as the three named test files and is fixed with them.

## Open risks

- The tmux-tier test file `internal/testkit/tmuxkit/tmuxkit_integration_test.go` cannot be run here; its change is a constant substitution checked by compiling.

## Acceptance criteria

- A launch on a tmux whose version line parses below 3.5 runs no `allow-set-title` `set-option` and logs no Warn about it.
- A launch with an unknown version still runs the `set-option`, and its failure still Warns and the launch continues.
- A parsed version at or above 3.5 runs the `set-option`.
- The floor cache logs the fallback once per path at Info and probes once.
- `toolfloor`'s check reports at-or-above for 3.5 and later, below for 3.4 and 3.4a, and unparsed for an unrecognised line.
- With a failing watch opener, a stop marker written during an idle cycle makes the watchdog exit on a stop request, and the trace names the fallback.
- The existing stop rows pass whether or not a watch opens.
- No test file named in the scope spells `"LYX_TRACE_ID"`.
