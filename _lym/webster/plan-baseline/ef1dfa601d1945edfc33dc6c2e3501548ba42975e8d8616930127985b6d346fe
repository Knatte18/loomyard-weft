# Card 5 — run-report-paused-by-adopt

**Create:**
- `plan:internal/adoptcli#runStatePausedByAdopt` -> `const runStatePausedByAdopt = "paused_by_adopt"`
- `plan:internal/adoptcli#ownerHaltResume` -> `func ownerHaltResume(status shedengine.Status) string`

**Delete:**
- `internal/adoptcli#runStateHeldFailedCheck`
- `internal/adoptcli#runStateWaitingForCheck`
- `internal/adoptcli#waitingLine`
- `internal/adoptcli#checkStateWords`
- `internal/adoptcli#checkStateEnded`
- `internal/adoptcli#checkStateNone`

**Edit:**
- `internal/adoptcli#reportInput`
- `internal/adoptcli#runReports`
- `internal/adoptcli#reportRuns` -> `func reportRuns(deps adoptDeps, here string, before, moved, paused, subjects []string) ([]runReport, error)`
- `internal/adoptcli#adopt`
- `internal/adoptcli#adoptDeps`
- `internal/adoptcli#hubDeps`
- `internal/adoptcli#hubDepsAs`
- `internal/adoptcli#world`
- `internal/adoptcli#newWorld`
- `internal/adoptcli#world.deps`
- `internal/adoptcli#heldUnderAlpha`
- `internal/adoptcli#TestRunReports`
- `internal/adoptcli#TestAdopt`
- `internal/adoptcli#TestAdoptFields`

**Uses:**
- `plan:internal/pauserecord#AdoptCaller`
- `internal/pauserecord#HoldCaller`

**Intent:**
D5's run report: the `waiting_for_check` and `held_failed_check` states become one `paused_by_adopt` state.

`runReports` gives a run `paused_by_adopt` when a live entry of its own scope carries `pauserecord.AdoptCaller` or the legacy `pauserecord.HoldCaller`.
The state keeps its place in the order, after `landed` and before `paused`.
Its line is `<slug>: paused by adopt[ at <row>]: `+"`lym tender resume <slug>`"+` lifts it`.
A run whose status is blocked with a reason opening `landingshed.OwnerHaltPrefix` gets `ownerHaltResume`'s clause added to its line.
The clause is `; then run `+"`lym loom resume`"+` in its pair`, since no green end resumes it any more (D8).
The `halted` line of an owner-halted run not under an adopt pause adds the same clause.
`ownerHaltResume` returns "" for any other status.

The check-state reading goes with the states: `reportInput` drops `Held` and `Started` and gains `Paused []string`, the slugs an adopt paused, which join the candidates as the held slugs did.
`checkStateOf`, `waitingLine`, `checkStateWords`, `checkStateEnded` and `checkStateNone` lose every reader.
`adoptDeps.CheckState`, the seam only `runReports` read, is dropped from the struct, from `hubDeps` and from the test world (`world.checkStates`).
`reportRuns` takes `paused` in place of `held` and `checks`; `adopt` passes its held slugs flattened, which card 6 replaces with the pause step's list.

`TestRunReports` (tier0) replaces the `HeldFailedCheck…` and `WaitingForCheck…` rows with rows naming the new state:
- an `adopt` entry of its own scope reads `paused_by_adopt`, naming its row and `lym tender resume t1`;
- a legacy `adopt-hold` entry reads the same;
- a released task under an `adopt` entry reads it with no row;
- an owner-halted run under an `adopt` entry names `lym tender resume t1` and `lym loom resume`;
- an owner-halted run with no adopt entry reads `halted`, naming `lym loom resume`;
- a run blocked for another reason reads `halted` with its line compared whole, so a stray `lym loom resume` clause fails it.
The `HeldFailedCheck…` rows, which set `w.checkStates` to `checkStateEnded` or `checkFailed`, are deleted with their state, not rewritten.
`LandedOutranksHeld` keeps its intent over an `adopt` entry.
`heldUnderAlpha` writes an `AdoptCaller` entry and fills `Paused`.
`TestAdopt` rows that set `checkStates` drop it.
`TestAdoptFields` (tier0) rewrites its `waiting_for_check` row to the `paused_by_adopt` state and its line.
`hubDepsAs` is where the dropped `CheckState` closure, and its `checkStateNone` and `checkStateEnded` reads, live.

**ImpactSummary:** the run report's state set and its input change inside adoptcli; the envelope's `runs` field keeps its shape, only its state values change.
