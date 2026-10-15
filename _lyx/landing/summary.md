# Type the orch start color once per session, not after every re-exec

The orch watcher typed `/color` into the orch pane every time it re-exec'd onto a new `lyx` binary.
`selfexec.Check` re-execs the watcher in place after each deploy, and the fresh process's `initCursor` set the start color pending unconditionally.
So the color was retyped minutes after a deploy, mid-work, into a session that already showed it.
The start color also wrote no `orch: injection typed` line, so these typings never showed in the watcher log.

## What changes

- `State` gains `color_strand` and `color_launched_at`: the strand and `LaunchedAt` the color was last typed for.
  Every color typing that lands records them, the start color and the reload sequence's color step alike.
  A held or failed typing records nothing.
- `initCursor` sets the start color pending only when that recorded pair differs from the bound session's strand and `LaunchedAt`.
  A re-exec on the same session types nothing.
  A new or adopted session gets a new strand and a new `LaunchedAt` through `ResetForFreshLaunch`, so it still gets its color.
- The start color logs `orch: injection typed` with `step=start-color`, like the reload steps.
- The color paragraph in `internal/orchengine/doc.go` describes the persisted record in place of the old memory-only mark.

## Design decisions

- `LaunchedAt` stands for the session start the color is keyed on.
  A state file from an older binary has no recorded pair, so the first watcher on the new binary types the color once and records it.
- The reload color step records the pair too.
  Without that, a session whose start color the color step took over (`typeStartColor` skips the start color in that case) would get it again on the next re-exec.
- The record is only written once a typing lands, so a failed color is retried by the next watcher that binds, as before.

## Tests

`internal/orchengine/watcher_test.go`:

- `TestWatcher_StartTypesTheColorOnTheFirstIdleTickOnly` now asserts that a restarted watcher on the same session types nothing, that a new `LaunchedAt` types the color once again, and that a failed typing is typed again after a restart.
- `TestWatcher_RestartMidColorStepTypesItOnceWhenIdle` asserts that a re-exec after the reload color step types no start color.
- `TestWatcher_HeldInjectionRecordsReasonLogsOncePerChangeWithinAHoldAndTypedLogsWait` asserts the `step=start-color` log line.
