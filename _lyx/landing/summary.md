# Type the orch color once per session, not after every re-exec

The orch watcher re-execs in place onto each new `lyx` binary, and the fresh process's `initCursor` set the start color pending unconditionally.
After every deploy it typed `/color` into the orch pane on the first idle tick, minutes later and mid-work, on a session that was already colored.
The start typing also wrote no log line, so these colors never showed in the watcher log.

Now the color is typed only when something can have reset it.
`State` in `internal/orchengine` records `ColorStrand` and `ColorLaunchedAt`, the strand and session launch the color was last typed for.
`initCursor` sets `colorPending` only when that pair differs from the bound strand and `LaunchedAt`, so a re-exec on the same session types nothing.
A new or adopted session gets a new strand guid and `LaunchedAt` through `ResetForFreshLaunch`, so it still gets its color.
The start color now logs `orch: injection typed` with step `start-color`, like the reload steps.

## Design decisions

- The pair is recorded on every color typing that lands, in the start color and in the reload sequence's color step, so a re-exec after a reload types no start color.
  The reload sequence itself is unchanged: a `/clear` or a compaction still retypes the color.
- A held or failed typing records nothing, so it is retried as before.
- The new `State` fields are plain JSON keys.
  A state file written before this change reads as never typed, so the first watcher on the new binary types the color once.
  A zero `LaunchedAt` from such a file still compares equal across re-execs.
- The watcher stays the only writer of the color, and nothing is typed unless the idle probe passed on the same tick.
- Trade-off: a color reset by something other than a new session, `/clear` or a compaction is no longer retyped by a re-exec.

The color paragraph in `internal/orchengine/doc.go` is updated to match.

## Tests

`TestWatcher_StartTypesTheColorOnTheFirstIdleTickOnly` now covers a restart on the same session typing nothing, a new launch typing once, and a failed typing recorded as nothing and retried.
`TestWatcher_RestartMidColorStepTypesItOnceWhenIdle` covers a re-exec after the reload color step typing no start color.
The held-injection log test also expects the `start-color` line.

Not run by hand: a real deploy against a live orch pane, to confirm that no color is typed after the re-exec.
