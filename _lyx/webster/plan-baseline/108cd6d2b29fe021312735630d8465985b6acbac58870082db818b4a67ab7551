# Card 6 — pane-too-short

**Edit:**
- `internal/shuttleengine/engine.go`
- `internal/shuttleengine/session.go`
- `internal/shuttleengine/session_test.go`
- `internal/shuttleengine/claudeengine/session.go`
- `internal/shuttleengine/claudeengine/session_test.go`
- `internal/orchengine/watcher.go`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/watcher_compact_test.go`
- `internal/orchengine/doc.go`
- `internal/orchcli/session.go`
- `internal/orchcli/smoke_live_test.go`
- `internal/orchcli/cli_test.go`
- `internal/reedengine/doc.go`

**Intent:**
Name a held cycle when the orch pane is too short to draw an input box (D16).

Shuttle: add a provider-neutral `IdleProbe` struct (`Idle bool`, `TooShort bool`) in `engine.go`, and a `SessionCycler.PaneTooShort(capture string) bool` method.
`Runner.SessionIdle` returns `(IdleProbe, error)`, filling `TooShort` from `PaneTooShort` only when the capture is not idle.
Claude's `PaneTooShort` reports a capture with no input-box caret whose line count (the pane's height) is below the rows an input box needs; declare that row count as a named constant beside the box-shape constants in `claudeengine/session.go`, derived from the box's rules, caret line and footer.
A busy pane of normal height is never too short.

Orch: `orchengine.Session.SessionIdle` returns `(shuttleengine.IdleProbe, error)`, and `runnerSession` delegates.
Every idle probe in the watcher goes through one helper: a probe reporting `TooShort` records `State.Stuck` as `orch pane too short for the idle probe; resize or use the larger client` (saved and logged once, as `markStuck` does), and the next passing probe clears that reason and only that reason.
This makes `State.Stuck` also carry an idle-phase hold reason; because it is set and cleared only by the probe's result, the transitions that already reset `Stuck` cannot hide it for longer than one tick.
Cycles and notice delivery stay held while the pane is too short, as for any failing probe.

Tests: `claudeengine/session_test.go` covers a short capture with no box (too short), a busy capture of normal height (not too short) and an idle one;
`shuttleengine/session_test.go`'s fake and assertions move to `IdleProbe`;
the watcher fakes in `watcher_test.go` and `watcher_compact_test.go` return `IdleProbe`, and a watcher test asserts a too-short probe records the stuck reason in phase idle and the next passing probe clears it;
`cli_test.go` asserts `lyx orch status` reports the reason under `stuck`.
`smoke_live_test.go`'s direct `runner.SessionIdle` call reads `.Idle`.

Docs: `internal/orchengine/doc.go`'s idle rules name the too-short reason; `internal/reedengine/doc.go` records, beside its existing `window-size latest` notes, why the policy is kept rather than changed or conditioned for the orch pane and that the window returns to the larger client's size at that client's next keystroke.

**ImpactSummary:** Changes the idle probe's return type through Runner, orchengine's Session seam and their fakes, and adds one SessionCycler method; stuck reporting in phase idle is new.
