# Card 16 — orch-start-waits-for-stopping-watcher

**Edit:**
- `internal/orchengine/watchloop.go`
- `internal/orchengine/state.go`
- `internal/orchengine/watchloop_test.go`
- `internal/orchcli/start.go`
- `internal/orchcli/start_test.go`
- `internal/orchengine/doc.go`

**Intent:** D11: `lyx orch start` waits for a stopping watcher instead of reading a dying one as live, so a `kill` followed by `start` spawns a fresh watcher (#390 restart race).

State, in `state.go`: add `WatcherStopping bool` (`watcher_stopping`), commented as set by a watcher that has received SIGINT or SIGTERM and not yet released `watch.lock`.
It is a state field, not config.

Watcher, in `watchloop.go`:

- `Watcher.Run`, once it holds the watch lock, clears the record together with the exit reason it already clears (`recordExit("")` becomes the one update that clears both).
- While `Run` loops, a goroutine bound to `Run`'s lifetime waits on `ctx.Done()` and, as soon as the context is cancelled, sets `WatcherStopping` under the state lock through `updateState`, so the record lands before the in-flight tick finishes and before the deferred lock release; it does nothing when `Run` returns first.
- Add `WaitWatcherGone(p Paths, bound, interval time.Duration, sleep func(time.Duration)) (live bool, err error)`: while the state records the watcher stopping and `WatcherLive` reports the lock held, sleep `interval` and probe again, up to `bound`; it answers whether a watcher still holds the lock at the end.
  With no stopping record it probes once and returns, so a healthy live watcher costs no wait.

Start, in `start.go`: replace the direct `orchengine.WatcherLive` call with `WaitWatcherGone(c.paths, 3*c.cfg.PollInterval(), c.cfg.PollInterval(), c.sleep)`, a fixed wait of three watcher poll intervals, then decide with `DecideStart` as usual.
Add a `sleep func(time.Duration)` field on the receiver, set to `time.Sleep` in the pre-run, so tests inject it.
When the bound passes with the watcher still live and still recording stopping, start reports the watcher live as today, and the envelope adds `"watcher_stopping": true` and the hint `the watcher is stopping; run "lyx orch start" again once it has exited`.
Document the wait in the verb's `Long` help and in the watcher section of `internal/orchengine/doc.go`, beside `watch.lock`.

Bound: the wait happens only while the stopping record is present, and lasts at most three poll intervals; past it start behaves as before and names the way forward, a lyx verb, per `PATTERN-no-denied-recovery`.

Tests:

- `watchloop_test.go`: cancelling `Run`'s context while a tick is blocked in the session fake persists `WatcherStopping` before `Run` returns and before the lock is free; a new `Run` clears it at start; `WaitWatcherGone` returns not live once a lock holder recording stopping releases within the bound, returns live past the bound, and probes exactly once with no stopping record.
- `start_test.go`, through the cobra `start` verb with the harness's fakes and an injected `sleep` that releases a held `watch.lock` (taken through `internal/lock` on the harness's paths) after the first probe: with the state recording the watcher stopping and a live strand, start spawns a watcher; with the lock never released, start reports the watcher live with `watcher_stopping: true` and the hint, and spawns none.

**ImpactSummary:** Adds one orch state field, one watcher-loop goroutine and one probe helper used only by orch start; a healthy live watcher and every other orch verb are unaffected.
