---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/dobbyengine/passes.go:129
    summary: A pause-entry-covered paused run whose work reached its parent gets neither a pass nor a notice, and the Goal's second idea (lym adopt finishes landed runs) is not made; both are the writer's own unrecorded narrowing of the Goal.
    fix: Leave the code in place and defer this finding to the run's parent. The only sound change is the parent recording or refusing the narrowing with lym loom decision add, which lies outside the fixer's write surface.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/battencli/dobbyreads.go:185
    summary: The DobbyRuns read that sets ReachedParent, the only production source of the field, has no test; TestChildReachedParent covers childReachedParent, not the paused-only gate, the argument pair or the Warn-and-leave-unreached path.
    fix: Add a case to internal/beamcli/dobbypairanchor_test.go, beside the existing DobbyRuns tests. A paused batten status whose child already reached the parent asserts ReachedParent true; a running run asserts false; an unreadable judgment asserts false without an error from DobbyRuns.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/dobbyengine/doc.go:70
    summary: The first bullet of "The passes" still says a run gets a pass only while batten's state is running, which the reworded lines under it and drivable now contradict.
    fix: Reword the doc.go:70 bullet to say a run gets a pass while batten's state is running, or paused with its work already on its parent, and keep the cancel marker and in-flight clauses.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/battencli/dobbyreads.go:191
    summary: The Warn for an unreadable reached-parent judgment fires on every dobby tick for each affected paused run, so a persistent failure floods the log.
    fix: In DobbyRuns, log the Warn once per slug and failure text, or log at a Level that follows the package's loop-logging policy, and clear the memory when the judgment reads again.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/passes.go:186
    summary: The liftPending cleanup change from State != running to !drivable is pinned by no test; the only test reaching it uses a blocked run, which behaves the same under both conditions.
    fix: Add a drivePasses subtest in internal/dobbyengine/passes_test.go beside the one at line 521. A paused run with ReachedParent set is covered, lifted while a pass is in flight or held, and gives no stale full pass afterwards.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/beamcli/doc.go:135
    summary: A paused run is now a state in which a hand pass is overridden by dobby, but lym batten goto also leaves a run paused and no pause entry covers it, so a goto on a run whose child history holds a finished Finalize is resumed at the next probe tick.
    fix: Name goto beside batten pause in the sentence added at contracts/orchrole/orch-role.md rule 3, so the orch holds such a run with lym tender pause, or have the dobby read refuse a reached-parent verdict that rests on the child history alone when batten is not past Run-Shed.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/passes_test.go:37
    summary: No TestPlanPass row pins a paused reached-parent run with OwnedElsewhere or Held set, though the summary claims both follow a running run's rules and OwnedElsewhere gains new reach.
    fix: Add two rows to TestPlanPass, so that a paused ReachedParent run with OwnedElsewhere on a probe tick gives passFull and with Held gives passNone. Fold the OriginOwed and CancelMarked rows added in this diff into the table if they duplicate the running-run rows, since the check is state-independent.
  - id: F8
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabriccli/anchor.go:318
    summary: Operator-facing text still advises lym batten pause as the way to quiesce or abandon a run (anchor.go:318, pairteardown/claim.go:102, the batten pause help), which no longer holds a run whose work reached its parent.
    fix: In those three places point at lym tender pause <slug> for a run whose work already reached its parent, or state the exception in the lym batten pause help at internal/battencli/cli.go:261.
---

### [MEDIUM] Covered runs and adopt are left out of the Goal

**Location:** internal/dobbyengine/passes.go:129, _lym/landing/summary.md:21-25
**Issue:** The Goal's Idea says dobby finishes a landed run even when it is paused, or at least sends the orch a notice naming it, and that `lym adopt` finishes landed runs as part of the takeover.
The incident's pause came either from the adopt hold or from the orch pausing the adopted runs.
Under the diff an adopt hold (`run.Held`) and a `lym tender pause` entry both leave such a run with no pass, and `summary.md` states the covered run gets no notice either.
Only a bare `lym batten pause` is repaired, so neither named cause of the incident gets the finish or the notice.
The summary records both narrowings as design decisions, but no `lym loom decision add` entry carries them, so they are unrecorded parent corrections.

### [MEDIUM] DobbyRuns' ReachedParent read has no test

**Location:** internal/battencli/dobbyreads.go:185-196
**Issue:** `ReachedParent` reaches production only through this closure.
`TestPlanPass` feeds the field by hand and `TestChildReachedParent` calls `childReachedParent` directly, so removing the read, passing the wrong `found` argument or dropping the paused gate fails no test.
The summary names `TestChildReachedParent` as the pin for the judgment, which holds for the judgment but not for the wiring.

### [LOW] Stale first bullet in the dobbyengine package doc

**Location:** internal/dobbyengine/doc.go:70
**Issue:** The bullet says a run gets a pass only while batten's state is running.
The two lines after it, and `drivable`, say a paused run whose work reached its parent gets one.

### [LOW] The reached-parent Warn repeats on every tick

**Location:** internal/battencli/dobbyreads.go:191
**Issue:** `DobbyRuns` runs on every dobby tick and the Warn has no dedup, so a failure that persists, such as a git read failing inside the Finalize past-merge check, writes one line per tick per paused run.
The same package dedups its repeated read failures with a Fail and Recover pair at runnotice.go:161.

### [LOW] The liftPending cleanup change is untested

**Location:** internal/dobbyengine/passes.go:186
**Issue:** The subtest at passes_test.go:521 uses a blocked run, which makes the old and the new condition behave the same.
Reverting the line to `run.State != runStateRunning` fails no test.

### [LOW] goto leaves a run paused with no pause entry

**Location:** internal/beamcli/doc.go:135
**Issue:** The orch role says `lym batten pause` and `lym batten goto` both leave the run paused, and the diff adds the exception for `pause` only.
A goto on a run whose child history already holds a finished Finalize reads as reached through `childReachedParent`, so dobby resumes it at the next probe tick and undoes the goto.

### [LOW] Missing TestPlanPass rows for a paused run owned elsewhere or held

**Location:** internal/dobbyengine/passes_test.go:37
**Issue:** The summary says a held run and a run owned elsewhere follow the same rules as a running one, and no row pins either for a paused run.
`OwnedElsewhere` has new reach, since a paused reached run owned elsewhere now gets a full pass on each uncovered probe tick.
The `PausedReachedParentCancelMarkedGivesNone` row repeats the state-independent check that `CancelMarkedGivesNone` already pins, and the `OriginOwed` row pins only the no-reopen kind that `OriginOwed` alone already forces.

### [NIT] Operator-facing pointers to lym batten pause

**Location:** internal/fabriccli/anchor.go:318
**Issue:** `anchor.go:318`, `pairteardown/claim.go:102` and the `lym batten pause` help in `battencli/cli.go` still present `lym batten pause` as the hold, with no mention that a run whose work reached its parent is finished by dobby instead.

## Rejected

- code-affected-set finding 2 (doc.go:136, run notices only for running runs): the lens itself found the text still true, since a paused reached run gets no run notice.
- Loop on a pass that exits 0 and leaves the run paused: the risk is bounded by the probe interval and named in `summary.md`, and a consumed `pause_requested` clears on the persist that records paused, so the incident's own pause does not repeat.
- Declared PATTERN changes: the directive declares none and the diff touches no `PATTERN.md` or `pattern/` file.
