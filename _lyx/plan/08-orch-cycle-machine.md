# Card 8 — orch-cycle-machine

**Create:**
- `internal/orchengine/watcher.go`
- `internal/orchengine/watcher_test.go`

**Uses:**
- `internal/orchengine/state.go`
- `internal/orchengine/prompt.go`
- `internal/orchengine/config.go`
- `internal/shuttleengine/engine.go`

**Intent:**
Implement the watcher's decision core: one poll of D9's idle check and D10's persisted cycle, driven entirely through a seam so every acceptance case is an untagged unit test.

`Session` is the seam, satisfied in production by card 11's adapter over `shuttleengine.Runner` and reed:
`StrandAlive(guid string) (bool, error)`, `ReadEvents(guid string, offset int64) ([]shuttleengine.Event, int64, error)`, `ContextTokens(turnEnd shuttleengine.Event) (int, bool, error)`, `SessionIdle(guid string) (bool, error)`, `Send(guid, text string) error` and `ClearSession(guid string) error`.
`Clock` is `Now() time.Time`.
`NewWatcher(session Session, cfg Config, paths Paths, stencilsDir string, clock Clock) *Watcher` builds one.
`(*Watcher).Tick() (done bool, err error)` runs one poll, loading state at the start and saving it before every side effect.

The watcher's read cursor and in-memory newest-event record follow the overview's `events-cursor` decision.
On its first tick the cursor starts at `PhaseEventsOffset` in a non-idle phase and at `LastInjectionOffset` in `idle`.
Every event read then lies past the relevant injection by construction, so a pre-clear handoff turn end is never evaluated for usage after `/clear` (D7).

Every tick first calls `StrandAlive` on `State.Strand`.
False records `WatcherExit` (the strand is gone or its pane is dead) and returns `done`.
An error is returned without touching state, because reed failing to answer says nothing about the session.
The tick then reads new events and, for each new turn end (`EventStop` or `EventWaiting`) read in `idle`, takes a `ContextTokens` reading into `LastContextTokens`/`LastContextKnown`.
The newest event and the clock time it was first read are kept in memory.

Phase `idle` injects only when all of D9's conditions hold:
the newest event read is a turn end, never `EventAsk`;
at least `IdleGrace()` has passed since it was read;
and a fresh `SessionIdle` probe passes.
It also needs a trigger: a known reading at or over `Threshold()`, or a pending cycle request (D14).
An unknown reading never triggers.
When all hold, it persists `handoff-requested` (strand, entry time, entry offset = cursor, `PendingHandoff` from `NewHandoffPath`), clears the cycle request, then `Send`s `RenderHandoffInstruction`.
Any unmet condition skips the poll with nothing written.

Phase `handoff-requested` waits for the pending file to exist and be non-empty and for a turn end past the entry offset (D11).
Both together record `LastHandoff`, persist `clearing`, then call `ClearSession`.
An `EventAsk` past the entry offset, or `HandoffTimeout()` expiring from the entry time, aborts: back to `idle` with no `/clear`, and `LastAbortReason` recorded.
The next attempt waits for the next qualifying idle turn end.

Phase `clearing` waits for a passing `SessionIdle` probe, or for the timeout.
Either way it persists `resuming`, increments `CycleCount`, then `Send`s `RenderResumePrompt` for `LastHandoff`.
Startup-ready classification is not used here, since a pre-clear pane shows it too.

Phase `resuming` returns to `idle` at the first turn end past its entry offset.
It sets `LastInjectionOffset` to the resuming entry offset and takes that turn end's reading, which is the first post-cycle reading, from the new transcript.
On timeout it returns to `idle` with `LastAbortReason` recorded.

D10's restart rules run once, on the first tick of a watcher that loads a non-idle phase whose `PhaseStrand` equals `Strand`:

- `handoff-requested` or `resuming`: a turn end past the entry offset, or a failing `SessionIdle` probe (a turn in progress), means the earlier injection landed, so keep waiting.
  An idle pane with no such turn end means it did not, so re-`Send` the phase's stencil — the same `PendingHandoff` path for a handoff.
- `clearing`: re-call `ClearSession`, then wait as above.

A persisted non-idle phase whose `PhaseStrand` differs from `Strand` is reset to `idle` with the abandoned phase recorded, injecting nothing.

`watcher_test.go` drives `Tick` against a fake `Session`, a settable clock and a temporary `Paths` and stencils directory.
Each case asserts the calls the fake recorded and the persisted state:

- below threshold → no injection
- above threshold and idle → handoff injected, `handoff-requested` persisted before the send
- newest event `EventAsk` → no injection
- `SessionIdle` false → no injection, and the next poll injects once it passes
- a newer event inside the idle grace → no injection
- unknown usage → no injection
- a manual cycle request below threshold → cycle runs and the request is cleared
- handoff file written but no turn end → no `/clear`
- handoff turn ended but no file → no `/clear`
- handoff timeout → `idle`, no `/clear`, abort recorded
- `EventAsk` during the handoff → abort, no `/clear`
- handoff written and turn ended → `/clear`, then the resume prompt naming that file
- the idle probe never passing after `/clear` → resume prompt after the timeout
- the resume turn never ending → `idle` after the timeout, abort recorded
- a pre-clear turn end read after `/clear` → no usage reading from it and no second cycle
- a restart in each non-idle phase with the injection landed → no duplicate send
- a restart in each non-idle phase with it not landed → exactly one re-send
- a persisted phase for a different strand → reset, abandoned phase recorded, nothing injected
- the strand gone → `done`, with `WatcherExit` recorded
