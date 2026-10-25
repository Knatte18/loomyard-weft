---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/dobbyengine/passes.go:planPass
    summary: the full pass at a pause lift is unreachable for an ordinary covering pause, so a lift gives a continue pass
    fix: In planPass, in the uncovered tail after the covered block, return full (passFullNoReopen when run.OriginOwed) when trigger.Lift is set, before the continue-pass fallthrough. Change the test row CoveredLiftGivesFull in passes_test.go so it plans with covered=false and Lift true, the way triggerFor produces it, and keep the OwnedElsewhere lift rows.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/boardengine/board.go:116
    summary: migrateDraftStatus runs after the mutation, so the first write to an unmigrated board re-drafts a task that same write released
    fix: In boardCriticalSection, run migrateDraftStatus over the loaded entries before fn mutates the store, or have it skip an entry whose status the write set to nil on an entry that was already a task in loaded; applyDraftRule keeps running after the mutation. Add a TestDraftMigration row where a null-status release is the first write to a board without draft_status and the task stays released.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: design
    location: internal/dobbyengine/passes.go:spawn
    summary: liftPending is cleared only when a pass spawns, so a lift seen while the run is not running stays pending and triggers a lift pass at an unrelated later time
    fix: In drivePasses/triggerFor, delete liftPending[slug] when the run is listed but planPass returns passNone for a reason other than an in-flight pass (state not running, cancel marker), or when the run is no longer listed; keep it while a pass is in flight.
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: consistency
    location: internal/battenshed/deps.go:117
    summary: the ChildArbiterStop On and Timeout doc comments still say they are read from the child's loom config, but the arbiter reading now resolves them from shed.yaml plus the recipe's override
    fix: Reword the comments on ChildArbiterStop.On and ChildArbiterStop.Timeout to say the value comes from the arbiter keys in shed.yaml with the recipe's module override.
    origin: lens:code-affected-set
---

### [MEDIUM] Full pass at a pause lift is unreachable

**Location:** `internal/dobbyengine/passes.go` `planPass`, `triggerFor`
**Issue:** `triggerFor` sets `liftPending` only when the run was covered and no longer is, so `planPass` sees `Lift` with `covered=false`. Outside the `OwnedElsewhere` branch, the uncovered tail returns a continue pass for a run at Run-Shed with no full pass owed. The `covered && Lift` branch, which the test row `CoveredLiftGivesFull` pins, cannot occur in production. D4 and D12 give the lift a full pass, and the acceptance criteria give the lift of an owner pause, as by `lyx adopt`, a full pass. A continue pass skips the origin reconcile and the owner re-judgement. Dobby's own owner pause lift is unaffected, since the `OwnedElsewhere` branch handles it.

### [MEDIUM] Draft migration can undo a release on the first write

**Location:** `internal/boardengine/board.go:116`, `internal/boardengine/draft.go` `migrateDraftStatus`
**Issue:** The migration runs after `fn` and marks every open task with an empty status, so a `set-status` release that is the first write after the upgrade leaves the task `draft` again and the operator's release is lost without a message. Later writes are unaffected, since the key is then set.

### [LOW] Stale liftPending

**Location:** `internal/dobbyengine/passes.go` `spawn`, `triggerFor`
**Issue:** `liftPending` is deleted only in `spawn`. A lift seen while the run is not `running` or carries a cancel marker stays set, and a pass later triggers off it at an unrelated time.

### [LOW] ChildArbiterStop comments name the old config source

**Location:** `internal/battenshed/deps.go:117`
**Issue:** `On` and `Timeout` are documented as read from the child's loom config. `arbiterstop/holdreading.go` now reads them through `sheddriver.ArbiterLimits` and the recipe override.

## Rejected

- Lens code-tests F-T1 (no test asserts the yield Debug log): the fork opened no test body and says so; a log level is not a behaviour the repository tests elsewhere, and the PATTERN line is pinned by the code path itself.
- Lens code-tests F-T2 (respawn-cap reset and third-episode trip have no named row): speculative, the fork saw only names and says the cap test may cover the reset.
- Lens code-tests F-T3 (start_test.go and starts_test.go sizes, overlapping sleep-watch subtests): no duplicated assertion shown, and the subtests sit in different files for the loop and the planner.
- Lens code-tests F-T4 (re-exec, fswatch fallback, `max_runs` rows without a named test): the fork lists them as probably covered and did not read the bodies.
- Lens code-pattern-fit F-A (no test pins that batten does not link tree-sitter): the fork's own `go list -deps` showed no tree-sitter package in `battencli` or `battenshed`, so the claim holds, and a missing pin is not a defect in the diff.
- Own check: `refreshWatched` moved from `check` to the start of `Call`, as D4 asks for the marker write once per pass with the dobby's pid, so it is not a departure.
- Own check: the three declared PATTERN entries land with the declared text in `PATTERN.md`, and `pattern/PATTERN-owner-machine.md` carries the matching section, so no declared-change finding is raised.
