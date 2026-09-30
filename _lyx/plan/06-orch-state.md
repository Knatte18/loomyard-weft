# Card 6 — orch-state

**Create:**
- `internal/orchengine/state.go`
- `internal/orchengine/state_test.go`

**Uses:**
- `internal/state/state.go`
- `internal/orchengine/config.go`

**Intent:**
Declare the orch module's persisted state and its told paths: the record D10's state machine persists before each side effect, D14's `status` reports, and D14's `cycle` request writes.

`Paths` is a plain struct of absolute paths, filled by `orchcli` (card 10) and never derived here, per the Told-Geometry Invariant:
`Dir`, `StatePath`, `StateLockPath`, `WatchLockPath`, `StartLockPath`, `CycleRequestPath`, `HandoffsDir` and `WatchLogPath`.
The overview's `orch-dir-layout` decision fixes their layout.

`Phase` is a string type with the four D10 values `idle`, `handoff-requested`, `clearing` and `resuming`.
`State` (JSON) carries:

- `Strand` — the orch strand's guid, written by `start` (see the `orch-strand-identity` decision).
- The persisted phase: `Phase`, `PhaseStrand` (the strand the phase belongs to), `PhaseEnteredAt`, and `PhaseEventsOffset` (the events-file position at entry — "a turn end after the injection" means one read past it).
- `PendingHandoff` — the handoff path the current `handoff-requested` phase asked for.
- `LastHandoff` — the last completed handoff, set only once its phase completes (D6, D10).
- `LastInjectionOffset` — the events position of the watcher's last completed cycle; only turn ends past it are evaluated for usage (D7).
- `LastContextTokens` and `LastContextKnown`.
- `CycleCount` — cycles that reached `/clear`.
- `LastAbortReason`.
- `WatcherExit` — why the last watcher exited, empty while one runs.

Functions:

- `LoadState(p Paths) (State, error)` over `state.ReadJSON`, returning a zero `State` with `Phase` `idle` when the file is absent.
- `SaveState(p Paths, s State) error` over `state.WriteJSON`, creating `p.Dir` first.
- `RequestCycle(p Paths) error`, `CycleRequested(p Paths) (bool, error)` and `ClearCycleRequest(p Paths) error` over the request file; clearing an absent request is not an error.
- `NewHandoffPath(p Paths, now time.Time) string` — a new file under `HandoffsDir` named from a UTC timestamp with second precision, so every cycle gets its own kept file (D12).
- `ResetForFreshLaunch(s State, strand string) State` — D10's fresh-launch rule: when the persisted phase is not `idle` it records the abandoned phase in `LastAbortReason`;
  it then sets `Phase` to `idle`, `Strand` to `strand`, zeroes `PhaseEventsOffset` and `LastInjectionOffset` (a new run has a new events file), and clears `PendingHandoff` and `WatcherExit`.
  `LastHandoff`, `CycleCount` and the context reading survive.

`state_test.go` covers: an absent state file loading as `idle`;
a save/load round trip of every field;
the cycle request's write, probe and clear, including clearing an absent request;
two `NewHandoffPath` calls a second apart differing and both sitting under `HandoffsDir`;
and `ResetForFreshLaunch` over an `idle` and a non-idle state — the non-idle one recording the abandoned phase, both zeroing the offsets and keeping `LastHandoff`.
