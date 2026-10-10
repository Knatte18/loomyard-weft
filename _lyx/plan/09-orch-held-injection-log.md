# Card 9 — orch-held-injection-log

**Create:**
- `plan:internal/orchengine#heldReason` -> `func heldReason(probe shuttleengine.IdleProbe) string`
- `plan:internal/orchengine#Watcher.logHeld` -> `func (w *Watcher) logHeld(st *State, probe shuttleengine.IdleProbe) error`
- `plan:internal/orchengine#Watcher.logTyped` -> `func (w *Watcher) logTyped(st State, step string)`
- `plan:internal/orchengine#reloadStepName` -> `func reloadStepName(step int) string`

**Edit:**
- `internal/orchengine#Watcher`
- `internal/orchengine#Watcher.initCursor`
- `internal/orchengine#Watcher.probeIdle`
- `internal/orchengine#Watcher.logIdleStateDisagreement`
- `internal/orchengine#Watcher.tick`
- `internal/orchengine#Watcher.tickCompacting`
- `internal/orchengine#Watcher.typeReloadStep`
- `internal/orchengine#Watcher.enter`
- `internal/orchengine#paneTooShortReason`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine#SessionSignalSessionStart`

**Intent:** The watcher logs why each step waited and when it was typed (D8), over the readiness reading the probe now carries (D7).
`heldReason` returns the probe's `Reason`, else `paneTooShortReason` for a probe reporting `TooShort`, else a fixed text for a not-idle probe with no reason (the pane fallback).
`probeIdle` stays the one door: after a probe that is not idle it calls `logHeld`, which records the reason in `State.Stuck` (saved when it changes) and logs `orch: injection held` at Info with the phase, the reload step name, the reason and the time since the phase or step began, once per change of reason in the manner of `markStuck`; a passing probe clears a `Stuck` the hold wrote, and only that, through the same memory of the last written reason that replaces the `paneTooShortReason`-only rule.
`Watcher` gains the hold memory: the last held reason, when the hold began, and the last session-start time logged; `initCursor` resets it and `enter` resets the hold's start.
`typeReloadStep` calls `logTyped` after each successful typing, which logs `orch: injection typed` at Info with the same fields and the wait's length, and clears the hold memory; `reloadStepName` names a step for both logs.
`tick` logs `orch: compaction boundary found` at Info with the boundary's time when a fresh boundary is read at a turn end, and `tickCompacting` logs the same when a qualifying boundary completes a requested cycle, so the delay from boundary to reload is readable from `watch.log` alone.
`logIdleStateDisagreement` keeps comparing the probe with the state, and also logs the hook's session-start event once per new `RunSessionState.SessionStartAt`, with its time, at Info.
`watcher_test.go`: the fake probe gains a scripted reason; rows assert a held reload step records the reason in `Stuck` and logs once while the reason is unchanged, a changed reason logs again, the typed step logs its wait and clears `Stuck`, a `TooShort` probe still records `paneTooShortReason`, and the boundary time is logged; the #481 row scripts the compaction boundary found with the probe idle on the next tick and asserts the reload typed on that tick.
`doc.go`'s idle rules and reload sections state the held and typed logs and that the probe is the readiness reading.

**ImpactSummary:** The watcher's probe door gains logging and a wider `Stuck` rule, three private helpers are added, and no seam or persisted field changes shape.
