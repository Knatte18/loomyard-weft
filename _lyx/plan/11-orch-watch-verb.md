# Card 11 — orch-watch-verb

**Create:**
- `internal/orchcli/watch.go`
- `internal/orchcli/session.go`
- `internal/orchcli/watch_test.go`

**Edit:**
- `internal/orchcli/cli.go`

**Uses:**
- `internal/orchengine/watcher.go`
- `internal/orchengine/watchloop.go`
- `internal/shuttleengine/session.go`
- `internal/shuttleengine/run.go`

**Intent:**
Add `lyx orch watch`, the detached daemon D4 describes, and the production adapter that satisfies `orchengine.Session`.

`session.go` declares `runnerSession`, adapting the receiver's `*shuttleengine.Runner` and the `strandOps` seam to `orchengine.Session`:
`ReadEvents`, `ContextTokens`, `SessionIdle` and `ClearSession` delegate to card 1's `Runner` methods, and `Send` to `Runner.Send`.
`StrandAlive` reads `strandOps.Strands()` and answers whether the guid is tracked with a live pane.
Each call that reaches tmux logs at `Debug` with the strand guid, per Live-Substrate Spawn Observability's polling-probe rule.

`watch.go` builds the `watch` subcommand, marked `Hidden` because `start` launches it and an operator never needs to (D14); it still carries a `Short`.
It loads state and refuses on the envelope when no strand is recorded, since a watcher with nothing to watch should say so in `watch.log` rather than spin.
Otherwise it builds `orchengine.NewWatcher(runnerSession{…}, cfg, paths, stencilsDir, realClock)`.
It runs `Run` with a context cancelled on SIGINT/SIGTERM (`signal.NotifyContext`) and `time.Sleep`.
At exit it prints one envelope naming the recorded `WatcherExit`, which lands in `watch.log` because `start` redirects the daemon's output there.
`ErrWatcherRunning` is reported as a success envelope with `already_running: true`, since a racing second spawn finding the first one alive is the intended single-instance outcome.

Register it in `cli.go`'s subcommand list.

`watch_test.go` covers, with a fake `strandOps` and no real reed:
`runnerSession.StrandAlive` for a live, a dead-pane and an untracked guid;
`watch` refusing when state records no strand;
and `watch` reporting `already_running` when the test holds the watch lock.

**ImpactSummary:** Adds one hidden subcommand to `orchcli`'s own tree; nothing outside the module changes.
