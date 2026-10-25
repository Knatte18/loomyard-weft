```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. Case run for each function or branch (✓ correct, ✗ finding).**

- `applyDraftRule`:
  - ✓ New task with nil status gets `draft`.
  - ✓ Existing task with nil status, write leaves it untouched: stays nil.
  - ✓ Existing task with status `running`, write sets nil (a release): stays nil.
  - ✓ Existing task with status `running`, write sets `""`: becomes `draft`.
  - ✓ Existing task with nil status, write sets `""`: becomes `draft`.
- `migrateDraftStatus`: ✗ F1.
- `FabricReplay.Apply` and `parseDraftStatus`:
  - ✓ Empty file reads false.
  - ✓ The commit's key is adopted into the store and marks the replay applied.
  - ✓ The replay does not migrate, as the record says.
- `Shed.stepLocked` Yield case and `Run`:
  - ✓ Persists nothing and reports state running with `Next` set to the current producer.
  - ✓ `Run` returns `RunYielded`.
  - Not verified: any pre-switch code in `stepLocked` that validates or persists the output pointer. Only the new case is in the diff.
- `StepEnvelope`: ✓ `continue` is false for a yield.
- Go driver `run`: ✓ A yield (`OK`, state running, outcome `yield`) sets `ExitYield` before `settleStop`, so no rule row, repair or budget applies.
- `InnerRun.Call` with memory:
  - ✓ `loadMemory` error returns early with no save.
  - ✓ The deferred `saveMemory` runs on Yield, Stuck and error.
  - ✓ A child state that differs from the remembered episode state returns a state-change Stuck before any arm runs.
  - ✓ The next pass begins with the new state.
- `InnerRun` respawn cap:
  - ✓ Zero `respawnStatusTime` resets the count on first sight.
  - ✓ `respawns` becomes 1 after the first respawn.
  - ✓ At 3 it logs and spawns nothing.
  - ✓ A non-running state zeroes the count.
  - ✓ A status write after `respawnStatusTime` resets it, as D4 specifies.
- `holdSpawnWhileCovered`:
  - ✓ Covered: yields with the hold reason.
  - ✓ Owner stands and the probe is due: runs the owner check; if it clears `ownedBy` and no pause covers, returns nil and the spawn goes ahead.
  - ✓ Pause flag set: returns the pause-requested Stuck.
  - Minor, no finding: after the owner check, `lastProbe` is set, so that pass's `once` sees `probeDue` false and skips the probe work.
- `foldOwnerTrigger`:
  - ✓ Fires on release or on a probe check while an owner stands, and no longer on a sleep.
  - ✓ Deliberate per D4.
- `checkChild`: ✓ The mtime short-circuit is safe, because the Call decodes the status at entry.
- `reviveDeadDriver`: ✓ The pid comparison is replaced by presence of the marker, which fits one process per pass.
- `planPass`: ✗ F2. The other branches I ran were correct:
  - non-running state, cancel marker, held driver lock or pass in flight → none
  - owned elsewhere, probe tick, uncovered → full
  - covered with a full pass owed → none
  - covered, at Run-Shed → continue
  - origin-owed → full with no reopen
- `drivePasses`, `recordOwed`, `triggerFor`, `collectExits`, `passExited`:
  - ✓ Map deletion while ranging is safe in Go.
  - ✓ A spawn failure counts as a failing pass with exit 1.
  - ✓ A refused continue pass owes a full pass.
  - ✓ The covered and owned-elsewhere exits count as no failure.
  - ✓ The third failure with the same text posts one notice, recorded only after it went out.
  - ✓ An exit-0 full pass clears the owed flag only when no origin-owed marker stands.
- `Run`, `loop`, `tick`, `sleep`, `nextWait`:
  - ✓ A second dobby on the held lock returns `ErrDobbyRunning`.
  - ✓ A re-exec (same pid) owes nothing new.
  - ✓ A nil or failed watch falls back to the timer, since a select on a nil channel blocks.
  - ✓ The timer doubles up to 60s and any event or in-flight run resets it.
  - ✓ A probe interval under one second is floored.

## Findings

**F1 — MEDIUM, design.** `internal/boardengine/board.go:116`, `draft.go:15-27`
- Summary: `migrateDraftStatus` runs after the mutation, so on the first write to an unmigrated board it also marks `draft` any task that same write just released.
- Run: a `set-status {"status":null}` release on a board without `draft_status`. The release leaves a nil status, the migration marks it `draft`, and the release is lost.
- Evidence: `migrateDraftStatus(store)` is called after `fn` and before `applyDraftRule`.
- Fix direction: migrate the loaded entries before `fn` runs, or exclude entries the write touched.

**F2 — MEDIUM, design.** `internal/dobbyengine/passes.go` (`planPass`, the uncovered tail), `triggerFor`
- Summary: the full pass at a pause lift is unreachable for an ordinary covering pause.
- Run: a run is covered, then the pause lifts. `triggerFor` sets `liftPending` only when `wasCovered && !run.Covered`, so `planPass` is called with `covered=false`. If the run is not `OwnedElsewhere`, has no `FullPassOwed` and is at Run-Shed, it falls through to `passContinue`.
- The `covered && trigger.Lift → full` branch, pinned by test row `CoveredLiftGivesFull` (`passes_test.go:45`), can't happen in production, because `Lift` is only set when the run is no longer covered.
- Spec: D4 and the acceptance criteria ("no full pass bar the one at the lift"; the lift of an InnerRun owner pause "gets the run its full pass").
- Effect: after `lyx adopt` lifts InnerRun's own owner pause, the run gets a continue pass. That skips the origin reconcile and owner re-judgement the spec says the lift pass must run.
- Fix direction: in the uncovered path, return `full` when `trigger.Lift`, and correct the test row to uncovered.

**F3 — LOW, design.** `internal/dobbyengine/passes.go` (`liftPending`)
- Summary: `liftPending` is cleared only in `spawn`. A lift that arrives while the run is not `running`, or is cancelled, stays pending. The run later gets a lift-triggered pass at an unrelated time.
- This is harmless but stale.

Not verifiable from my context: the `apiRevive` logic in `notice.go`, the exit-code mapping of `ExitYield`, and the `spawnConfirmed` pid semantics. The shown diff only implies per-pass confirmation.

## Tool calls
1. `grep Lift internal/dobbyengine/passes_test.go` and `grep RunAwaiting|RunYielded` over `internal`. I needed the test rows that pin the lift behaviour, and the other consumers of `RunOutcome`. The only non-test consumer outside `shedengine` is a comment in `loomcli/arm.go`.
