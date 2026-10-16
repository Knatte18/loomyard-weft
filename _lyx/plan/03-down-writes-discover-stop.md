# Card 3 — Down writes a stop request when it ends the server

**Create:**
- `plan:internal/reedengine#DiscoverStopMarker` -> `const DiscoverStopMarker = "stop"`
- `plan:internal/reedengine#Engine.writeDiscoverStop` -> `func (e *Engine) writeDiscoverStop()`
- `plan:internal/reedengine#TestDown_WritesTheDiscoverStopOnlyWhenItEndsTheServer` -> `func TestDown_WritesTheDiscoverStopOnlyWhenItEndsTheServer(t *testing.T)`
- `plan:internal/reedengine#TestTouchDiscoverSignal_TruncatesAPendingStop` -> `func TestTouchDiscoverSignal_TruncatesAPendingStop(t *testing.T)`

**Edit:**
- `internal/reedengine#Engine`
- `internal/reedengine#Engine.Down`
- `internal/reedengine#DownResult`
- `internal/reedengine/doc.go`

**Uses:**
- `internal/reedengine#Engine.touchDiscoverSignal`
- `internal/reedengine#Engine.ensureServerGoneLocked`
- `internal/fsx#AtomicWriteBytes`
- `internal/reedengine#newTestEngine`
- `internal/reedengine#installFakeTmux`
- `internal/reedengine#fakeTmux.answer`
- `internal/testkit/logcapture#Capture`

**Intent:**
D1 and the engine half of D3.
`DiscoverStopMarker` is declared in `watchdog.go` beside `DiscoverSignalFileName`: the fixed content of the discover signal file that asks the hub's watchdog daemon to stop, spelled once for the teller and the daemon.

`Engine.writeDiscoverStop` writes `DiscoverStopMarker` as the whole content of the told `e.geom.DiscoverSignalPath` through `fsx.AtomicWriteBytes`, a temp file renamed onto the path, so the daemon's watch never sees the file created empty before the marker lands: a plain `os.WriteFile` raises a create event first, and a daemon consuming on it would read an empty file, remove it and lose the stop.
An empty path (standalone, which tells none) writes nothing, and a write failure logs one Warn naming the socket and path and never fails `Down`.
`touchDiscoverSignal` is unchanged: its truncating write turns a pending stop back into an ordinary discover, so a racing `up` wins.

`DownResult` gains `EndedServer bool`, true when `Down` took its server-teardown branch (no session left, or the listing errored).
In that branch `Engine.Down` calls `writeDiscoverStop` before `kill-server`, so the stop is written whether or not the server then confirms gone; on a `serverErr` `Down` still returns its error.
A `Down` that leaves a sibling session writes nothing and reports `EndedServer` false.
`PATTERN-told-geometry` holds: `Down` writes only the told path and learns nothing of the daemon's lock.
Every `Engine.Down` caller, `internal/pairteardown` included, gets the stop with no change of its own.

`Engine` gains the test seam `ensureServerGone func(serverPID int) error`, nil meaning `ensureServerGoneLocked`, which `Down` calls through; production never sets it.
It exists because the not-confirmed-gone exit needs a socket holder that survives `SIGKILL`, which no hermetic test can stage otherwise.

`TestDown_WritesTheDiscoverStopOnlyWhenItEndsTheServer` drives `Down` on `newTestEngine` with a `DiscoverSignalPath` under the test hub's scratch directory and the fake tmux answering a dead pane listing, so no real process is reaped, as `TestDown_ListsEveryWindowsPanesOverACorruptState` does; it captures logs with `logcapture.Capture`.
Rows:

- last session (`list-sessions` empty): `Down` returns nil, the file's content is exactly `DiscoverStopMarker`, `EndedServer` is true.
- listing errors (`list-sessions` answers an error): the same.
- sibling live (`list-sessions` answers another session): no signal file, `EndedServer` false.
- no path told (`DiscoverSignalPath` empty), last session: no `DiscoverSignalFileName` under `fabricengine.HubScratchDir(hub)`, the hub's directory entries are unchanged, `EndedServer` true.
- told path names an existing directory, last session: `Down` returns nil, `EndedServer` true, and exactly one Warn naming the path is logged.
- server not confirmed gone (`ensureServerGone` returns an error), last session: `Down` returns that error and the file's content is exactly `DiscoverStopMarker`.

`TestTouchDiscoverSignal_TruncatesAPendingStop` writes the stop through `writeDiscoverStop`, calls `touchDiscoverSignal`, and asserts the file exists and is empty, so a cold boot after a stop and before the daemon consumes it turns the stop back into a discover.

`internal/reedengine/doc.go`'s discover-signal paragraph adds that `Down`, when it ends the server, writes `DiscoverStopMarker` into the file atomically to ask the daemon to stop, that a cold boot's touch truncates it back to an ordinary discover, and that `DownResult.EndedServer` reports the teardown.

**ImpactSummary:** `Engine.Down` gains one best-effort atomic file write on its server-teardown branch, one result field and one nil-by-default test seam; its callers (`internal/reedcli`'s `down` verb, `internal/pairteardown`) compile unchanged.

**Commit:** 3: Down asks the hub watchdog to stop when it ends the server
