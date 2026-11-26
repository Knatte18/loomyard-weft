# Let a test wait for an entered watchdog session's goroutines to stop

`TestEnterSession_DrivesTheToldTmux` in `internal/reedcli` now and then failed at cleanup with `TempDir RemoveAll cleanup: ... directory not empty`.

## Cause

`enterSession` starts the session's OOM watch, and the name-repair loop unless the worktree says `watchdog: off`, with bare `go` statements.
The returned `watchedSession` held only a `cancel`, so nothing could wait for those goroutines to return.
`watchPaneOOMKills` reads the strands at once, through `Engine.Status`.
That read takes the reed op lock, which first creates the worktree's `.lym` directory and its `reed.lock` file.
The test's cleanup only cancelled, so it could return while the first read was still writing under the hub fixture,
and `t.TempDir`'s removal then found the directory refilled.

## Change

- `watchedSession` carries `running`, a `*sync.WaitGroup` over every goroutine `enterSession` starts, which reaches zero once each has returned after `cancel`.
- `stopWatchedSession`, a test helper, cancels the session and waits on `running`, failing the test if the goroutines are still running 30s after the cancel.
- The two tests that call `enterSession`, `TestEnterSession_DrivesTheToldTmux` and the flip-to-off case in `watchdog_integration_test.go`, clean up through it.
  Their cleanups are registered after the fixture's temp directories, so they run before those directories are removed.

## Design decisions

- The daemon's loop still only cancels a departed or reaped session and never waits on `running`.
  The OOM watch's read blocks on the worktree's reed op lock, so waiting there would let one long reed operation stall discovery.
  Only tests read `running`, so the field adds no production behaviour.
- The name-repair goroutine is counted in `running` as well as the OOM watch, so a session's wait covers every goroutine started for it.

## Merge danger

**Door:** two-way
**Blast radius:** reed
