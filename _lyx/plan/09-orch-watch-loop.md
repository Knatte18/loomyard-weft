# Card 9 — orch-watch-loop

**Create:**
- `internal/orchengine/watchloop.go`
- `internal/orchengine/watchloop_test.go`

**Uses:**
- `internal/orchengine/watcher.go`
- `internal/orchengine/state.go`
- `internal/lock/lock.go`

**Intent:**
Wrap `Tick` in the daemon loop D4 describes: single-instance per prime, polling until the strand is gone.

`ErrWatcherRunning` is the sentinel for a second instance.
`WatcherLive(p Paths) (bool, error)` probes the watch lock with `lock.TryAcquireWriteLock`: acquired means no watcher, so it releases and reports false.
Held means a watcher is live.
`status` and `start` both read watcher liveness through it.

`(*Watcher).Run(ctx context.Context, sleep func(time.Duration)) error` creates `Paths.Dir`, then takes the watch lock with `lock.TryAcquireWriteLock` and returns `ErrWatcherRunning` when another process holds it.
It holds the lock for the whole run and clears `WatcherExit` in state on entry.
It then calls `Tick` every `PollInterval()` until `Tick` reports `done` or `ctx` is cancelled.
A cancelled context records `WatcherExit` as stopped by signal.

A `Tick` error is logged at `Warn` and the loop continues, since reed or a stencil read failing once says nothing about the session.
Consecutive errors are capped by count, per Live-Substrate Spawn Observability's retry clause, because most ticks spawn tmux through reed.
On reaching the cap `Run` records `WatcherExit` naming the last error and returns it.
The cap is a named constant sized so the default poll interval gives about a minute of consecutive failures.
A successful tick resets the count.

`sleep` is a parameter so tests never sleep for real.

`watchloop_test.go` covers, with card 8's fake `Session` and a no-op sleep:
`Run` returning `ErrWatcherRunning` while the test holds the watch lock;
`WatcherLive` reporting true while the lock is held and false after release;
`Run` returning once the fake strand disappears, with `WatcherExit` recorded;
a context cancelled mid-run recording the signal exit;
and a fake that errors on every tick stopping `Run` at exactly the cap, while one successful tick between errors resets the count.
