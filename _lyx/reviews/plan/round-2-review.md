---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/06-driver-loop-to-sheddriver.md:84
    summary: "The arbiter-only restart row names no stop that reaches the arbiter; the harness's blocked restart fixture sends such a stop to the tackler"
    fix: >-
      In card 6, rewrite the `TestGoDriver_Settle` row that pins the arbiter restart read so it names its input:
      the restarted run's status is awaiting at the escalation producer, `Arbiter.EscalationPending` reads true,
      `OpenAttempts` returns an arbiter directory and no tackler directory, and `AttemptEnvelope` is scripted to return the saved stop envelope's path.
      The row asserts the calls are `AttemptEnvelope` and then the arbiter, with no step before them.
      Add one sentence saying a blocked restart fixture reaches the arbiter only through an applied tackler attempt, which already has its row and reads the envelope from the tackler attempt, so it never reaches `AttemptEnvelope`.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/06-driver-loop-to-sheddriver.md:52
    summary: "HandBackDeps.Park is made optional with nil parking nothing, an unbounded skip of the park marker that no recipe needs and D2 does not allow"
    fix: >-
      In card 6, delete "nil parks nothing" from the `HandBackDeps` bullet and list `Park` with the required hand-back seams (`OrchRecorded`, `Now`), so an unwired `Park` is a wiring fault like any other required seam.
      D2 makes the park marker part of every hand-back, and loom and batten both wire it.
      The harness's `Park` fake already writes the marker, so no test row changes.
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/13-batten-drives-its-run.md:58
    summary: "Card 13 says batten's status, run-lock, pause and pause-hold reads are batten's own, but battencli has no pause-hold and the other three are inline, and no test pins that every required seam is wired"
    fix: >-
      In card 13, replace "The status, run-lock, pause and pause-hold reads are batten's own" with the sources `driverDepsFor` builds them from:
      `ReadStatus` is `state.ReadJSONNoMkdir[shedengine.Status]` over `c.shedPaths.StatusPath` and `c.shedPaths.StatusLockPath`;
      `RunLockHeld` is a `lock.TryAcquireWriteLock` probe of `c.shedPaths.LockPath` that releases at once, as `battenPreStepUnmarked` does;
      `Pause` is `hubgeom.PauseGeometry(c.location)` and `pauserecord.ReadCoverage` for the slug, as `refuseCovered` does;
      `PauseHold` is `pauserecord.WaitUncovered` over the same geometry and slug, which battencli does not have today.
      Then add to `TestBattenDriverDeps` an assertion that every required `Deps` seam card 6 lists is non-nil (`Step`, `Goto`, `ReadStatus`, `RunLockHeld`, `SaveEnvelope`, `NewReportPath`, `Pause`, `PauseHold`, `Tackler`, `ConsumeTackler`, `OpenAttempts`, `Now` and the hand-back's `Notify`, `OrchRecorded` and `Park`), because a nil `PauseHold` panics only at the first re-step or tackler spawn and no run test reaches it.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/04-tackler-call-to-sheddriver.md:133
    summary: "Three TestTacklerPrompt rows render a loom, a darn and a batten recipe line, but sheddriver cannot import those lines, so the rows repeat one substitution that TestTacklerFor and TestBattenDriverDeps already pin per recipe"
    fix: >-
      In card 4, keep the row that asserts the rendered body (no "arbiter", the stop-kind list, the reading order, the write rule) and the row that refuses an empty `Recipe` or `GotoRows` naming the marker, and drop the three rows that render a loom, a darn and a batten line.
      Replace them with one row that renders a told recipe line and asserts the line is filled under "The run".
      Say in the card that card 8's `TestTacklerFor` pins the loom and darn lines and card 13's `TestBattenDriverDeps` pins `Tackler.Recipe` for batten, so the acceptance criterion's three recipes stay covered.
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/06-driver-loop-to-sheddriver.md:56
    summary: "ArbiterOutcome carries 'resume or not' but the moved code also prints the arbiter's verdict word and role in the stop report, and names neither without importing loomengine"
    fix: >-
      In card 6, add to the `ArbiterOutcome` field list a `Verdict` string and a `Role` string, copied by loom's wiring from `loomengine.ArbiterEnvelope.Verdict` and `loomengine.ArbiterRole`.
      Say the moved `arbitrate` fills the arbiter attempt note's `Role` and `Verdict` from them and tests `Resume` for the resume branch, so `internal/sheddriver` names neither `loomengine.ArbiterRole` nor `loomengine.ArbiterVerdictResume`.
      The stop report's attempt lines then stay byte-identical to today's.
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/plan/13-batten-drives-its-run.md:7
    summary: "battenCLI.driverDepsFor takes a release func that batten never uses, since batten wires no hand-over seam"
    fix: >-
      In card 13, change the Create target to
      `plan:internal/battencli#battenCLI.driverDepsFor` -> `func (c *battenCLI) driverDepsFor() (sheddriver.Deps, error)`,
      and have `driveRun` ignore the `release` that `sheddriver.DriveVerb` hands its callback.
      A run of `lyx loom validate-plan --overlay` with this one change reports nothing the baseline `{}` run does not.
---

### [LOW] The arbiter-only restart row names no input that reaches the arbiter

**Location:** `_lyx/plan/06-driver-loop-to-sheddriver.md:84`

**Issue:** The focus asked whether a test pins the restart of a halted run whose only open attempt is an arbiter attempt, once `resumeOpenAttempt` reads it through `AttemptEnvelope`.
Card 6 does add such a row, and the seam keeps `loomengine` out of the moved code, so the read itself is carried.
But the row says only "calls the arbiter before any step".
In today's `decideDrive` (`internal/loomcli/driverules.go`) a stop reaches the arbiter only through an awaiting stop with `EscalationPending`, or through `ResumeReturned`, which needs an applied tackler attempt, and that attempt also fills the tackler directory.
The harness's `restartAttempt` fixture in `drive_test.go` is a blocked stop, with `openApplied` the only route to the arbiter, and `resumeOpenAttempt` reads the envelope path from the tackler attempt first.
A blocked fixture with an arbiter directory alone goes to the tackler, so the row as written cannot pass, and the one reachable input never reaches `AttemptEnvelope`.

### [LOW] HandBackDeps.Park is optional with no bound on what a nil skips

**Location:** `_lyx/plan/06-driver-loop-to-sheddriver.md:52`

**Issue:** D2 lists the hand-back as writing the stop report, committing where the recipe commits, writing the park marker and queuing the notices, and it makes only the commit and the notices recipe-dependent.
Card 6 adds "nil parks nothing" to `Park`, and then gives the optional seams a bound (they only remove rows or agents from the table) that does not cover this one: a hand-back with no marker leaves a run that `lyx loom start` and `lyx loom resume` read as a driver not yet parked, a transient refusal that never clears.
Loom and batten both wire `Park`, so no caller needs the nil.
The attack-surface rule asks for what it can skip and what bounds it; the simpler fix is to not introduce it.

### [LOW] Card 13 names four seam sources that do not exist as stated, and pins none of the required seams

**Location:** `_lyx/plan/13-batten-drives-its-run.md:58`

**Issue:** The card says the status, run-lock, pause and pause-hold reads "are batten's own".
In `internal/battencli` the status read and the run-lock probe sit inline in `battenPreStepUnmarked`, the pause read sits in `refuseCovered` and in `wire`'s `Pause` closure, and nothing waits on a pause: `pauserecord.WaitUncovered` is called only from `shuttleengine` and `loomcli`.
`Pause` and `PauseHold` are required seams in card 6 and `holdPause` calls `PauseHold` before every re-step and tackler spawn, so a `driverDepsFor` that leaves it nil panics on the first stop it settles.
`TestBattenDriverDeps` asserts the optional seams are nil and says nothing of the required ones, and the run tests never hold a pause, so nothing reaches the gap.

### [LOW] Three recipe-line rows in TestTacklerPrompt repeat one substitution

**Location:** `_lyx/plan/04-tackler-call-to-sheddriver.md:133`

**Issue:** `internal/sheddriver` may not import `loomcli` or `battencli`, so the loom, darn and batten rows in `TestTacklerPrompt` render three literals typed into the test, and each exercises the same fill of `{{.recipe}}`.
The per-recipe line is already pinned where it is chosen: `TestTacklerFor` in card 8 asserts the darn and loom lines, and `TestBattenDriverDeps` in card 13 asserts `Tackler.Recipe` for batten.
One row that renders a told line, beside the row that refuses an empty value, covers the stencil behaviour once.

### [LOW] ArbiterOutcome leaves the arbiter's role and verdict words to an import the package may not have

**Location:** `_lyx/plan/06-driver-loop-to-sheddriver.md:56`

**Issue:** `arbitrate` in `internal/loomcli/drive.go` builds the arbiter attempt note with `loomengine.ArbiterRole` and the envelope's `Verdict`, and branches on `loomengine.ArbiterVerdictResume`.
Card 6 lists `ArbiterOutcome` as "resume or not, reason, question, goto, cause, attempt, attempt directory, verdict path" and says the boundary forces no other edits.
The note's `Verdict` and `Role` have no field to come from, and `sheddriver` imports no `loomengine`, so an implementer must invent a copy of the role and verdict words in `sheddriver` or add fields the card does not name.
The stop report prints both, and D2 and the constraints keep the attempt records unchanged.

### [NIT] driverDepsFor takes a release func batten never uses

**Location:** `_lyx/plan/13-batten-drives-its-run.md:7`

**Issue:** `sheddriver.DriveVerb` hands its callback a `release` so a hand-over can free the driver lock early, and only `HandOverSeam.ReleaseLock` consumes it.
Batten wires no hand-over seam, so `driverDepsFor(release func() error)` carries a parameter nothing reads.

## Rejected

- Card 6 lists `internal/loomcli/drivewiring.go` as a file while its Intent names `goDriverDepsFor`, `writeDrivePausedNote`, `newestAttemptsFor` and `driveCmd`.
  The member-glyph fix is blocked for `driveCmd`: `validate-plan --overlay` reports `redundant-file-target`, because the glyph resolves into `drive.go`, which the same card moves, before card 5 has run.
  The file target is the only admissible cover for that edit, so the form stays.
- `tacklerCall.settleFresh` calls `Snapshot` unconditionally today, but card 4 owns the nil-safety, states it in the Intent, and pins it with a `TestTacklerCall_SpawnPath` row that leaves `Snapshot` nil; card 6 and card 13 agree with it.
- The `driver_kind` refusal for the hub base layer is carried: card 8 has `battenshed.LoadConfig` refuse a resolved document holding the key, with a base-layer row in `TestLoadConfig`, and the local layer is refused by `configengine.ValidateLocal` (`internal/configengine/layers.go`).
- Card 13 lands the `refusal-spec.md` row for `battenDriveRefusal` with a way forward, a second `lyx batten run` as the input, and a `TestRunCmd_HeldRunLockRefusesNamingTheLockPath` row that holds the driver lock.
- Card 7 adds the `shed config key` refusals and card 8 lands their spec rows, shared with the loom, darn and batten rows; the plan lands as one task, so the rows ship with the refusals.
- No card depends on a later one: cards 4, 6, 8 and 13 reach only names earlier cards create, and the prose mentions of cards 12 and 13 in card 6 are forward notes, not dependencies.
- The `## verify:` section is in the template's order and `./...` covers every targeted package.
