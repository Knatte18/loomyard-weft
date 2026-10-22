# Card 11 — shuttle's gate loop holds its boundary while the run's session is covered

**Create:**
- `internal/shuttleengine#TestGate_HoldsItsBoundaryWhileCovered`
- `internal/shuttleengine#gateRigSettings`
- `internal/shuttleengine#gateRigOption`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.observePause`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#Run.handleGatedBoundary`
- `internal/shuttleengine#newGateRig` -> `func newGateRig(t *testing.T, cfg Config, events string, opts ...gateRigOption) *gateRig`
- `internal/shuttleengine#TestGate_WaitsForIdleWriter`
- `internal/shuttleengine#TestGate_UnsentRepromptRetries`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#Runner.targetSlug`
- `internal/shuttleengine#ErrPaused`
- `internal/shuttleengine#gateRig`
- `internal/shuttleengine#tuiReed`
- `internal/shuttleengine#hookClock`
- `internal/shuttleengine#coverScope`
- `internal/shuttleengine#coverHub`

**Intent:**
D3, and D2 for the gate re-prompt and pending send.
`observePause` reads the coverage of the slug D1 judges for the run's own session, `Runner.targetSlug` over the run's strand guid and its record's `StrandName`, falling back as it does, never the runner's slug alone; a run whose record holds the name takes no reed call for it, and for one that does not, `Run` gains a field caching the slug of the first successful lookup, since a strand's name is fixed for its life.
`Run` also gains a field marking a gated Done arrival that reached the boundary while covered.
In `Wait`'s tick, while `run.covered` reads true, a gated Done arrival records the boundary as reached (`gateAtBoundary` true, `startCleared` and `unsentReprompt` false, the rework spans ended as today) and marks the arrival held instead of calling `boundaryIdle` and `handleGatedBoundary`; the pending, unsent re-prompt and cleared-start branches are skipped, so no gate closure runs and no pending send goes out.
The first tick that reads uncovered with an arrival held clears the mark and evaluates the boundary as if the arrival had come then, through `boundaryIdle` and `handleGatedBoundary`.
The off-boundary `Expired` probes keep running under cover, since they send nothing, so an entry's own wait bound can still end a run; an `Expired` probe that fires under cover finalizes the run, whose final evaluation then waits in card 3's held Verify until the resume.
`handleGatedBoundary` holds a re-prompt send refused with `ErrPaused` exactly as one refused with `ErrSessionBusy`: no attempt spent, the memo cleared, the writer kept at the boundary with the re-prompt marked unsent; a pending send refused with `ErrPaused` already stays pending.
Bound: the hold only delays an evaluation; no verdict or attempt count changes because of it.
`doc.go`'s Pause section says the gate holds its boundary while covered, by the session's own slug, and that the expiry probes do not.

`newGateRig` gains optional settings, passed as trailing option functions so its two existing callers, `TestGate_WaitsForIdleWriter` and `TestGate_UnsentRepromptRetries`, compile and run unchanged: one replaces the rig's gate entry (a closure and an `Expired` probe of the row's own), one names the run record's `StrandName`.
`TestGate_HoldsItsBoundaryWhileCovered` is table-driven over `newGateRig`, its runner told a temp record, with the rig's clock hook adding or resuming the entry at set offsets:
a failing gate's Done arrival under a hub entry runs no gate closure and types nothing while covered, and after the resume evaluates once and re-prompts, spending one attempt;
the same under a slug entry of `p1` with the strand named `ly:p1:impl` and the runner's geometry slug `other`, which reads uncovered for the runner;
the same with the record holding no `StrandName` and reed's status naming the strand `ly:p1:impl`, which reaches the lookup, caches its slug and holds the boundary until the `p1` entry resumes;
a gate closure that adds a hub entry while it evaluates and fails has its re-prompt refused, spending no attempt and typing nothing, and the re-prompt goes out after the resume with one attempt spent;
a closure that answers pending with a `Send` text, with the entry added after the first pending send went out, sends no second pending send and runs the closure no more while covered, and resumes both after the resume;
a re-prompt left unsent by a busy pane when a hub entry is added is not retried while covered, and goes out after the resume with one attempt spent;
an entry whose `Expired` probe reports expired, over an events file seeded with a turn start and no turn end so the writer is off its boundary, finalizes the run while it is covered.

**ImpactSummary:** shuttle's Wait loop and its gate boundary handler change for covered runs only; a runner told no geometry never reads covered, so the existing gate tests run unchanged.
