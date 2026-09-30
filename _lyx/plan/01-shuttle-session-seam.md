# Card 1 — shuttle-session-seam

**Edit:**
- `internal/shuttleengine/engine.go`

**Create:**
- `internal/shuttleengine/session.go`
- `internal/shuttleengine/session_test.go`

**Uses:**
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/wait.go`
- `internal/shuttleengine/rundir.go`
- `internal/shuttleengine/fakes_test.go`

**Intent:**
Give shuttle the provider-invariant surface the orch watcher needs to read a live run and cycle its context, without widening `Engine`.

In `engine.go`, declare `SessionCycler`, an optional capability interface beside `Engine` (see the overview's `session-cycler-capability` decision):

- `ContextTokens(turnEnd Event) (tokens int, known bool)` — the provider's context usage as of the turn end `turnEnd` records; `known` false means usage could not be read, which a caller must never treat as over any threshold.
- `IdleSession(capture string) bool` — whether a pane capture shows the provider idle: its input box present and empty, and no turn in progress.
- `ClearSessionSequence() []PaneInput` — the key choreography that clears the live session's context.

Its doc comment states it is an optional capability: an `Engine` that also implements it lets `Runner`'s session methods work, and one that does not makes them return an error naming the missing capability.

In the new `session.go`, add four `*Runner` methods, each returning `r.toldErr` first like the existing entry points:

- `ReadEvents(guid string, offset int64) ([]Event, int64, error)`: resolve the run through `FindRun`, read its events file from `offset` through `readEventsFrom`, parse through `engine.ParseEvents`, and return the events plus the new offset.
  A partial trailing line stays unconsumed, a parse error returns the original offset, and an absent events file returns no events and the original offset, all matching `pollEventsTick`'s rules.
- `ContextTokens(turnEnd Event) (int, bool, error)`: delegate to the engine's `SessionCycler`, or error when the engine is not one.
- `SessionIdle(guid string) (bool, error)`: `requireLiveStrand`, then one `CapturePane`, classified by `IdleSession`.
  It logs the probe at `Debug`, per Live-Substrate Spawn Observability, since a capture spawns tmux.
- `ClearSession(guid string) error`: `FindRun` to confirm the guid is a shuttle run, `requireLiveStrand`, then `playInputs` over `ClearSessionSequence()`.
  It skips `requireReadyAgentPane` on purpose: right after a clear the pane may briefly show no ready marker, and the caller has already probed idleness itself.

`session_test.go` covers, against `fakeReed` and a test-local engine that embeds `fakeEngine` and adds scripted `SessionCycler` answers:
`ReadEvents` advancing past complete lines while leaving a partial line, returning the original offset for an absent events file, and erroring on an unknown guid;
all three cycler-backed methods erroring on a plain `fakeEngine`;
`SessionIdle` erroring on a dead strand and returning the scripted classification of the captured text;
and `ClearSession` playing exactly the scripted sequence into the fake reed.

**ImpactSummary:** Adds one interface to `engine.go` and new `Runner` methods; `Engine` itself and every existing implementer and fake stay unchanged.
