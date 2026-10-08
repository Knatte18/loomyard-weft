# Card 14 — orch-shadow-log

**Edit:**
- `internal/orchengine/watcher.go`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/doc.go`
- `internal/orchcli/session.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/shuttleengine#Runner.SessionState`
- `plan:internal/shuttleengine#RunSessionState`

**Intent:**
Log the orch session's state beside the orch watcher's idle probe (D5, second bullet), in shadow mode.

`orchengine.Session` gains `SessionState(guid string) (shuttleengine.RunSessionState, error)`, which `orchcli`'s `runnerSession` satisfies by delegating to `Runner.SessionState`; the orch session is a shuttle run with its own record (interactive, with a never-written sentinel output), so its turn ends read interactive `asking`.
`Watcher.probeIdle`, the one door every idle probe goes through, reads the state after a successful probe and logs `orch: session state disagrees with the idle probe` at Warn when the probe reads idle and the state is `busy`, or the probe reads not idle and the state is `idle-done`, `idle-stalled` or `asking`, naming the probe's answer, the state, its cause and since.
It logs once per disagreement and again only after either side changes; the last pair is kept in the watcher's memory, not in `State`.
A probe that reports `TooShort` is not compared, since its not-idle answer says nothing.
A failure to read the state is logged at Debug and changes nothing: `probeIdle` returns the probe and error it returns today, and no cycle, delivery or `State` write depends on the state (`PATTERN-orch-pane-single-writer`: nothing is typed).

Tests: `watcher_test.go`'s `fakeSession` answers `SessionState` from a scripted value or error; one table-driven test, with log capture, runs a tick whose idle probe disagrees with the scripted state and asserts one Warn across two ticks, no Warn when they agree or the pane is too short, and that the tick's saved `State` and the session calls it makes are identical with the state read failing.
`cmd/lyx/testdata/test-budget.yaml`: the `internal/orchengine` row rises by 1, for that test.

Docs: `orchengine/doc.go`'s list of `Session` methods gains `SessionState` and says the watcher only logs it.

**ImpactSummary:** orchengine.Session gains one read method, implemented by orchcli's runner adapter and the test fake; the watcher only logs it.
