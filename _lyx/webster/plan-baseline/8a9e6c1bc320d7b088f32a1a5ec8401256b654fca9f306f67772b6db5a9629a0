# Card 11 — the orch watcher waits out a held write instead of failing the tick

**Create:**
- `internal/orchengine#Watcher.heldTyping`

**Edit:**
- `internal/orchengine#State`
- `internal/orchengine#Watcher.deliverNotice`
- `internal/orchengine#Watcher.tickIdle`
- `internal/orchengine#Watcher.tickHandoff`
- `internal/orchengine#Watcher.startClearing`
- `internal/orchengine#Watcher.tickClearing`
- `internal/orchengine#Watcher.typeReloadStep`
- `internal/orchengine#Watcher.typeStartColor`
- `internal/orchengine#Watcher.typeColor` -> `func (w *Watcher) typeColor(st *State) (held bool)`
- `internal/orchengine#Watcher.startCompacting`
- `internal/orchengine#Watcher.tickCompacting`
- `internal/orchengine#Watcher.toIdle`
- `internal/orchengine#Watcher.enter`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/watcher_compact_test.go`
- `internal/orchengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine#ErrSessionBusy`
- `internal/orchengine#Watcher.probeIdle`
- `internal/orchengine#maxConsecutiveTickErrors`

**Intent:**
Treat a typed step that shuttle held, by its pre-write guard or by a foreign box after the paste, as nothing typed (decision D7), so the orch's own notices and cycle steps wait for the operator instead of failing ticks; `PATTERN-orch-pane-single-writer` is unchanged, since the watcher stays the only Go writer into the orch pane.

`State` gains `StuckBySend bool` (`json:"stuck_by_send"`): whether `Stuck` is a held idle-phase step's reason, which only a later send that passes clears; `enter` and `toIdle` reset it with `Stuck` and `StuckByHold`, and `probeIdle` and `logHeld` already leave a `Stuck` they did not write alone.

`heldTyping(st, step, err)` answers held false for a nil error or one that does not wrap `shuttleengine.ErrSessionBusy` (the caller returns it as today).
For a held error it logs at Info once per change of reason ("orch: injection held by the session", with the step and the reason, the first line of the error text) and answers held true; in the idle phase it also records that reason in `st.Stuck` with `StuckBySend` true and saves, unless a non-hold, non-send `Stuck` is already recorded.
Every typed step routes its error through it and, when held, returns nil leaving the phase, `PhaseInjected` and the notice file as they are, so the next tick retries the same text and the tick is not counted toward `maxConsecutiveTickErrors`:
- `deliverNotice` keeps the notice file, and on a send that passes clears a `StuckBySend` reason before removing the file;
- `tickIdle`'s handoff request, `tickHandoff`'s retype, `startClearing`'s and `tickClearing`'s `/clear`, `startCompacting`'s and `tickCompacting`'s `/compact`, and `typeReloadStep`'s plugins, skills, retry and pointer steps stay unconfirmed, which their next tick already retypes;
- `typeColor` returns held true for a held color (any other failure is still only logged, as display only), `typeStartColor` then keeps `colorPending`, and `typeReloadStep`'s color step does not advance to the plugins step.

Bound: a cycle phase held forever is still marked stuck by the existing `HandoffTimeout`, and an idle-phase notice held by a draft stays visible in `lyx orch status` through `StuckBySend` instead of flapping each tick (D7).

Tests:
- `watcher_test.go`: a fake session whose `Send` returns an error wrapping `ErrSessionBusy` leaves the notice file, records the reason in `State.Stuck` with `StuckBySend`, and returns no tick error; a passing idle probe on the next tick leaves that `Stuck`; a later `Send` that passes clears it and removes the notice; a held handoff request keeps `PhaseHandoffRequested` unconfirmed and retypes the same path on the next tick; a held `/clear`, a held skills step and a held color each retry on the next tick without advancing, with no tick error.
- `watcher_compact_test.go`: a held `/compact` keeps `PhaseCompacting` unconfirmed and is retyped on the next idle tick.
Raise `internal/orchengine`'s row in `cmd/lyx/testdata/test-budget.yaml` if a new top-level test is needed, and document the held-write rule in `internal/orchengine/doc.go`.

**ImpactSummary:** The watcher's typed steps gain one error check each and State one field; every caller is inside orchengine.
