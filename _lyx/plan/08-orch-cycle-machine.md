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
When all hold, it first renders `RenderHandoffInstruction` for a path from `NewHandoffPath`.
A render error is returned from `Tick` with the state untouched and the cycle request kept, so a broken stencil override never enters a phase that could only time out.
It then persists `handoff-requested` (strand, entry time, entry offset = cursor, `PendingHandoff`, `PhaseInjected` false), clears the cycle request, then `Send`s the rendered text.
Any unmet condition skips the poll with nothing written.

Phase `handoff-requested` waits for the pending file to exist and be non-empty and for a turn end past the entry offset (D11).
Both together first render `RenderResumePrompt` for the pending handoff, so the text the cleared session needs is known good before `/clear` runs.
A render error aborts: back to `idle` with no `/clear`, and `LastAbortReason` naming the resume stencil.
Otherwise it records `LastHandoff` and `PendingResume`, persists `clearing` with `PhaseInjected` false, then calls `ClearSession`.
An `EventAsk` past the entry offset, or `HandoffTimeout()` expiring from the entry time, aborts: back to `idle` with no `/clear`, and `LastAbortReason` recorded.
The next attempt waits for the next qualifying idle turn end, which the return-to-idle rule below enforces.

Phase `clearing` waits for a confirmed clear followed by a passing `SessionIdle` probe, or for the timeout.
While the clear is unconfirmed, the confirmation rule below re-calls `ClearSession`, because a clear that errored never reached the pane and a pre-clear pane passes the probe too.
On leaving it persists `resuming` with `PhaseInjected` false, increments `CycleCount` only when the clear was confirmed (card 6's "cycles that reached `/clear`"), then `Send`s `PendingResume`.
A timeout with no confirmed clear still moves to `resuming` (D10), and the resume prompt lands harmlessly in the uncleared session.
Startup-ready classification is not used here, since a pre-clear pane shows it too.

Phase `resuming` returns to `idle` at the first turn end past its entry offset.
It takes that turn end's reading, which is the first post-cycle reading, from the new transcript.
On timeout it returns to `idle` with `LastAbortReason` recorded, and marks the reading unknown (`LastContextKnown` false): the persisted reading predates `/clear` and says nothing about the cleared session.

Every return to `idle` — a completed resume, a resume timeout, or a handoff abort — persists `LastInjectionOffset` as the cursor the watcher has read through, and forgets the in-memory newest event.
Only a turn end read after the return can then qualify for the next injection, and a restarted watcher starts its `idle` cursor past everything the cycle saw.
Without this, a handoff turn end already read, or a stale pre-clear reading, could start a second cycle at once — into a freshly cleared session after a resume timeout, replacing `LastHandoff` with a handoff of an empty context (D7, D11).

Each non-idle phase's injection is confirmed through `PhaseInjected`.
It is saved false with the phase, before the side effect, and saved true once the `Send` or `ClearSession` returns nil.
An injection error is returned from `Tick` with the flag still false, so card 9's loop logs it and counts it toward its cap.
On every tick in a phase whose injection is unconfirmed, D10's landed check runs before the phase's own wait:

- `handoff-requested` or `resuming`: only a turn end past the entry offset means the injection landed, so mark it confirmed and keep waiting.
  A failing `SessionIdle` probe with no such turn end means keep waiting with the flag unchanged, so a later tick checks again.
  The probe also fails on a permission or select dialog, a draft or an unframable capture, so it is no evidence that a turn is running.
  A turn that really is in progress confirms through its own turn end, which is D10's "turn in progress → keep waiting".
  A passing probe with no such turn end means the injection did not land, so re-`Send` the phase's text — the handoff instruction re-rendered for the same `PendingHandoff` path, or `PendingResume` verbatim.
- `clearing`: re-call `ClearSession`.
  A repeated `/clear` after a completed handoff loses nothing, so this phase needs no landed check (D10).

A re-send happens at most once per tick and stops at the phase timeout; a persistently failing one stops at card 9's error cap, leaving the phase persisted for the next watcher.

D10's restart rules are this same check.
A watcher that loads a non-idle phase whose `PhaseStrand` equals `Strand` first saves `PhaseInjected` false, whatever it said, so the check runs on its first tick and on every later tick until a turn end confirms the injection or a passing probe triggers the re-send.
A restart in `clearing` therefore re-calls `ClearSession` unconditionally, and since `CycleCount` moves only on leaving `clearing`, the re-clear never counts the cycle twice.

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
- the resume turn never ending → `idle` after the timeout, abort recorded, reading marked unknown
- a pre-clear turn end read after `/clear` → no usage reading from it and no second cycle
- ticks after a resume timeout with an idle pane and no new turn end → no second handoff injected
- a handoff abort whose turn end was already read → no retry until a later turn end
- every return to `idle` → `LastInjectionOffset` persisted as the cursor read through
- a restart in `idle` after a resume timeout → the pre-clear handoff turn end is not re-read and no cycle starts
- a restart in `handoff-requested` or `resuming` with the injection landed → no duplicate send
- a restart in `handoff-requested` or `resuming` with it not landed → exactly one re-send
- a restart in `clearing` → exactly one `ClearSession`, then the resume prompt once the probe passes, and `CycleCount` incremented once
- a failing handoff render → no phase change, no `Send`, and the cycle request still present
- a failing resume render after the handoff turn ends → abort to `idle`, no `ClearSession`, abort reason naming the resume stencil
- a `Send` error on the handoff instruction, then an idle pane with no turn end → exactly one re-send on the next tick, naming the same `PendingHandoff` path
- a `Send` error on the resume prompt, then an idle pane with no turn end → exactly one re-send of `PendingResume` on the next tick
- a `Send` error on the resume prompt whose text did land (a turn end follows) → no re-send, and `idle` at that turn end
- a `clearing` timeout with the probe still failing, a `Send` error on the resume prompt, then ticks where the probe keeps failing and then passes with no turn end → no re-send while it fails, and exactly one re-send of `PendingResume` once it passes
- a `Send` error on the resume prompt while a turn is in progress (probe failing), followed by that turn's end → no re-send, and `idle` at that turn end
- a `Send` error on the handoff instruction behind a failing probe that later passes with no turn end → exactly one re-send, naming the same `PendingHandoff` path
- a restart in `resuming` with `PhaseInjected` persisted true, a failing probe on the first tick and a passing probe with no turn end on the next → exactly one re-send
- a `ClearSession` error, then success on the next tick → one move to `resuming`, `CycleCount` 1
- a `ClearSession` that errors until the timeout → `resuming` with the resume prompt sent and `CycleCount` unchanged
- a persisted phase for a different strand → reset, abandoned phase recorded, nothing injected
- the strand gone → `done`, with `WatcherExit` recorded
