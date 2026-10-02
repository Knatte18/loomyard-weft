# Card 12 — watchdog-name-repair

**Create:**
- `internal/reedengine/namerepair.go`
- `internal/reedengine/namerepair_test.go`

**Edit:**
- `internal/reedcli/watchdog.go`
- `internal/reedengine/naming_integration_test.go`

**Uses:**
- `internal/reedengine/overlay.go`
- `internal/reedengine/io.go`
- `internal/reedengine/lock.go`
- `internal/reedengine/watchloop.go`

**Intent:**
The per-hub watchdog repairs drift in both name mirrors on its own tick and logs every repair.

`namerepair.go` declares the provider seam reed stays blind behind (Shuttle Provider-Seam Invariant): `SessionNamer`, an interface with `SessionNameDrift(sessionID, workdir, want string) bool` (whether the provider session's current name differs from want; the implementation logs and answers false when it cannot read the name), `SessionIdle(capture string) bool`, and `RenameText(want string) string` (the literal text typed, before Enter, to rename the session).
reedengine declares it and never implements it; the Claude implementation and its wiring land in the next card.

`(e *Engine) WatchNames(ctx context.Context, namer SessionNamer) error` runs until ctx ends, ticking at a fixed `nameRepairCycle` of ten seconds, and each tick runs one `repairNames(namer)` pass under `withTryOpLock`, so a held op lock defers the tick rather than queueing behind it, as `reapplyLayout` does.
A pass on a session that is not up does nothing.
Otherwise it loads state and lists panes, then for each strand bound to a live pane:
- pane title: when `LivePane.Title` differs from `Strand.Name`, it runs `select-pane -T <name>` and logs `logger.Info("reed: repaired pane title", …)` with socket, session, guid, the old title and the name;
- session name, only when namer is non-nil and the strand carries a `SessionID` (only shuttle-launched agent strands do): when `namer.SessionNameDrift(s.SessionID, e.geom.PaneCwd, s.Name)` reports drift, it captures the pane and, only when `namer.SessionIdle` passes on that capture, types `RenameText(s.Name)` as a literal send-keys followed by Enter and logs `logger.Info("reed: repaired session name", …)`; a busy session logs at Debug and is retried next tick.
`SessionID` stays opaque to reed: it is handed to the provider unread.
The decision over strands and panes is a pure helper (`planTitleRepairs`) so it is tested without tmux.

`reedcli`'s `enterSession` starts `eng.WatchNames(ctx, nil)` on the same context as `eng.Watch`, so the existing cancel on departure stops both and a `watchdog: off` worktree gets neither.
A standalone run's in-process `Watch` is untouched, so its titles and session names are set at launch only.

`namerepair_test.go`, on the fake tmux with a fake `SessionNamer`: a drifted title is rewritten and logged, a matching one is left alone, a dead pane and an unbound strand are skipped;
a drifted session name on an idle pane is renamed once, with the typed text and the log line asserted, on a busy pane nothing is typed, a nil namer and an empty `SessionID` skip the session check, and a fake reporting no drift types nothing.
`naming_integration_test.go` (real tmux) gains: a title changed by hand with `select-pane -T` is restored by one `repairNames` pass, and the repair log line is captured.

**ImpactSummary:** Adds a second watchdog goroutine per watched worktree in the reed daemon and one exported seam type; the resize loop and every verb are untouched.

**Commit:** 12: reed watchdog: repair drifted pane titles and session names
