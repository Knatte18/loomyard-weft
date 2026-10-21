# Reed small fixes: trace-id constant in tests, allow-set-title warn noise, watchdog stop marker without inotify

`reed-small-fixes` · Task · bug, enhancement, reed, shuttle · running · Worktree-Create

## From note `trace-id-env-literal`

### Seen (Darn-Review of `gate-fewer-new-binaries`, 2026-10-20)

The run made its sites name the trace id variable through `logger.TraceIDEnv`, and the reviewer deferred three test files outside the finding's scope that still spell it `"LYX_TRACE_ID"`: `internal/reedengine/lifecycle_test.go`, `internal/shuttleengines/shuttleengines_test.go` and `internal/testkit/tmuxkit/tmuxkit_integration_test.go`.

### Idea

Those tests use `logger.TraceIDEnv`, so a rename of the variable cannot leave them testing a name lyx no longer sets. No effect on run time.
## From note `allow-set-title-warn-noise`

### Seen (2026-10-20)

- On the work laptop (Ubuntu 24.04, tmux 3.4), `lyx orch start --continue` printed one `WARN reed: could not turn off allow-set-title, the pane title may drift … invalid option: allow-set-title` per strand pane it launched.
- `internal/toolfloor/doc.go` keeps `allow-set-title` (tmux 3.5) optional on purpose: `internal/reedengine/spawn.go` Warns and continues, and `lyx reed list` reports title drift.
- The fallback is correct; the Warn is noise, repeated for every strand on every launch on a supported tmux.

### Idea

Detect the tmux version once (the floor check already knows it) and skip the `set-option` below 3.5, logging the fallback once per process at debug or info, instead of a Warn per pane.

## From note `reed-watchdog-inotify-fallback`

Imported from [issue #783](https://github.com/Knatte18/loomyard/issues/783).

Seen (orch, 2026-10-21, OSL-1046, seven runs in flight): TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping fails in all three stop subtests on every rerun of `lyx gate test ./internal/reedcli` ("marker consumed at call 0"), blocking the verify of create-target-transient, whose diff does not reach internal/reedcli.

Cause, per the darn writer: fs.inotify.max_user_instances is 128 and the user holds about that many instances (VS Code, konsole windows per run, lyx processes, other desktop apps). The watchdog consumes the stop marker only through its filesystem watch (internal/reedcli/watchdog.go, the `events` branch); when the watch fails to open, `events` is nil and the marker is never read.

Expected: the watchdog falls back to reading the stop marker on its timer cycle when the watch fails to open, and says so in its log, so a full inotify table neither breaks stops in production nor fails the test. The test should not depend on a free inotify instance, or should report the exhaustion as its failure cause.

Workaround: raise fs.inotify.max_user_instances (sudo sysctl) and rerun the verify.
