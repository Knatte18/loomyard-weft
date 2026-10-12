# Card 6 — shuttle-agent-waits

**Create:**
- `plan:internal/shuttleengine#AgentWait` -> `type AgentWait struct`
- `plan:internal/shuttleengine#ReadAgentWaits` -> `func ReadAgentWaits(cfg Config, anchorPath string, engine Engine, now time.Time) ([]AgentWait, error)`
- `internal/shuttleengine/agentwait_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine#ReadSessionStates`
- `internal/shuttleengine#ReadWaitMarkers`
- `internal/shuttleengine#WaitMarker.Held`

**Intent:**
Add the one reading of a pair's agents that the parent lock's idle rule (D7) needs, so the loop and batten read waiting the same way.
`AgentWait` is `StrandName`, `Waiting` and `Busy`.
`ReadAgentWaits` returns one entry per run under the run-directory root of `anchorPath` whose record reads running and whose waiting process is alive, the runs `ReadAgentActivity` reads.
A run is `Waiting` when its own wait marker is live and is the held turn end (`WaitMarker.Held`), or when its session state is `asking` or `idle-stalled`.
A run is `Busy` when its session state is `busy` and it is not waiting.
Every other state is neither.
The wait marker is read per run directory, so a held marker names its own run; the session state comes from the same fold `ReadSessionStates` uses, at `now`.
An absent root returns no entries and no error, and an unreadable run record is skipped and logged, as `ReadAgentActivity` does.
Provider specifics stay under `claudeengine` (`PATTERN-shuttle-provider-seam`): the function reads only the generic session-state names.

`agentwait_test.go` writes run directories with records, wait markers and session signals and asserts one table: a held turn end, an asking session and a stalled session read waiting; a busy session reads busy; a done session reads neither; a run whose waiter pid is dead is absent; an absent root reads empty.

**ImpactSummary:** one new read-only function over files shuttle already writes; no existing reader changes.

**Commit:** 6: shuttle: read each live run's waiting and busy state
