---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/03-run-watch-engine.md:84
    summary: The done-run clear holds only while a watch entry owes a line, so a done run whose run_watch level has never read is cleared with its entry and its landed line is never judged, unlike the landing notice, which holds the clear for an unread run past Land.
    fix: "In card 3's Entry upkeep paragraph, add that clearDoneRuns also leaves a run while RunWatch.Level is set and the run has no level read yet (no watchLevels entry this dobby), so the run waits for a read as an unread landing-notice memory makes it wait; add a TestClearDoneRuns row whose Level fails on every tick, where the done run is not cleared, and then reads `stops`, where the landed line posts and the run clears on the next tick."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/03-run-watch-engine.md:61
    summary: The card never says StatusSeen and ChildSeen stay unchanged after a failed ReadChild, so an implementation that advances them misses the child change behind the failure until the next status write, and a child halted at a stop writes nothing more; the planned row does not reach the next tick.
    fix: "In card 3 step 3, state that StatusSeen and ChildSeen are set only after a ReadChild that returns no error (found true or false), and that a failed read leaves both as they were so the next tick reads the child again; in the Memory paragraph, change 'of the last child read' to 'of the last successful child read'; extend the row 'a failing ReadChild keeps the child part last read and posts nothing for it' so that on the next tick, with StatusModified and ChildStatusModified unchanged and ReadChild recovered, ReadChild is called and the child's stop posts one line."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/03-run-watch-engine.md:59
    summary: A run ahead of Land copied as land_notice on, then left out of one tick's list, loses its landing-notice memory (rememberRuns drops an unlisted run that owes nothing) and is next seen past Land, where the landing notice never remembers it, while the watch keeps its true copy and skips the landed line, so the orch hears neither line.
    fix: "In card 3 step 2, add that on each tick the watch clears an entry's LandNotice copy (sets it nil) when State.LandNotices holds no memory of the slug and the run was not listed this tick, since rememberRuns then dropped the landing memory, so a run first seen again past Land is never skipped; add a TestRunWatch row: land_notice on is copied ahead of Land, the run is unlisted for one tick, then listed landed, and the watch posts its landed line."
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/03-run-watch-engine.md:58
    summary: Step 1 skips a run with no level read yet whole, step 2's copy included, but D3 copies the land_notice reading while the run is ahead of Land once that reading exists, whatever the watch level, so a run whose run_watch first reads after Land gets both the landing notice and a watch line though land_notice was on.
    fix: "In card 3, move step 2 (the copy) ahead of step 1's skip, or say in step 1 that the skip leaves the copy step running, so a run with no level read yet still copies State.LandNotices[slug].Requested while it is not PastLand; add a TestRunWatch row: the level fails through the ticks ahead of Land with land_notice on, reads after the run is landed, and posts no watch line for the landed reading."
  - id: F5
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/01-run-watch-setting.md:14
    summary: "Card 1 plans a new TestCheckLocalValues because no test calls CheckLocalValues, but TestPrepareRunSettings (internal/configreg/runsettings_test.go:82-104) already reaches it through configreg's CheckValues (internal/configreg/runsettings.go:223-224) with refusal rows for beam pull_request, land_notice and autonomy, so the new test duplicates covered behavior."
    fix: "In card 1, drop the Create line `plan:internal/dobbyengine#TestCheckLocalValues` and its Tests bullet; add `internal/configreg#TestPrepareRunSettings` under Edit, and in Tests give it two rows: a beam run_watch of rows is admitted into a beam local document (wantDocs beam run_watch: rows), and a beam run_watch of all is refused at the run settings (wantRefusal run_watch, wantModule beam)."
  - id: F6
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/plan/06-gate-hold-engine.md:56
    summary: Card 6 caps the gate-hold line at 20 descendants and 20 marked processes with gateHoldProcessMax, but D6 and its acceptance criterion name every other marked process and each child PID, and no Added after Discussion entry licenses the cap.
    fix: "Either drop the Create line `plan:internal/dobbyengine#gateHoldProcessMax`, the cap sentence in the Intent and the test row 'a holder whose tree holds more than 20 descendants names 20 and ends with ; and <n> more', so the line names every descendant and marked process as D6 says; or ask the parent to record the cap as an Added after Discussion entry and keep the card as it is."
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/03-run-watch-engine.md:22
    summary: "Card 3 lists internal/dobbyengine/doc.go under Extends, but its new clearDoneRuns hold makes the done-run-clear bullet (internal/dobbyengine/doc.go:185-191) false, since that bullet says a run is cleared once neither its landing notice nor its board-tidy count is owed; fixing it rewrites existing text."
    fix: "In card 3, move `internal/dobbyengine/doc.go` from Extends to Edit, and add to the doc.go sentence of the Intent that the done-run-clear bullet also names a watch line still owed; also have the clearDoneRuns doc comment name the owed watch line."
  - id: F8
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: _lym/plan/01-run-watch-setting.md:30
    summary: "The board upsert help in internal/boardcli/cli.go:587 (storeVerbs) says beam.pull_request, beam.land_notice and beam.autonomy set the task's per-task beam settings, which run_watch makes false, and no card lists it."
    fix: "In card 1, add `internal/boardcli#storeVerbs` under Edit and `internal/boardcli/doc.go` under Uses, and add an Intent sentence that the run_settings help line names beam.run_watch beside the three keys."
  - id: F9
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: _lym/plan/04-run-watch-wiring.md:7
    summary: docs/overview.md:850 lists what dobby sends the nearest live orch, its own notices and the dobby notice lines of a running child, and names neither the run watch lines nor the gate-hold notice, which is no running-child condition.
    fix: "In card 4, add `docs/overview.md` under Edit and an Intent sentence that the dobby paragraph names the [run watch] lines and the gate-hold [dobby notice] among what dobby sends the orch."
  - id: F10
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/05-gate-holder-marker.md:28
    summary: A nested gate run inside an inherited slot takes no slot but rewrites LYM_GATE_TEST_PID to its own PID for every process it starts (internal/gatecli/run.go:154-156,172), so an orphan of a nested tier carries a marker PIDsWithEnv(holder.Marker) never matches, and the card states no such bound.
    fix: "In card 5's Intent, add the bound that a process started by a nested gate run inside the holder's slot carries the nested run's marker, so the gate-hold notice does not name it as a marked process outside the tree."
  - id: F11
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/03-run-watch-engine.md:85
    summary: When the state record reads but does not write, the watch step ends without recording the landed line it owes, and clearDoneRuns on the same tick reads no owed line and clears the run, so the landed line is lost.
    fix: "In card 3, add that a tick whose watch step failed to write the state record runs no done-run clear (a daemon flag the clear checks), so the run stays until a later tick records its line."
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/06-gate-hold-engine.md:75
    summary: The failing-LoadConfig row calls d.gateHolds directly because startReady fails the tick on a failed LoadConfig, but gateHolds runs before startReady in the scheduling block, so tick reaches the guard and the stated reason is false.
    fix: "In card 6, drive the failing-LoadConfig row through probe ticks like the other rows and drop the sentence about startReady and the direct call."
  - id: F13
    severity: NIT
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/01-run-watch-setting.md:43
    summary: "The Intent has LoadConfig's doc comment name run_watch but not its two new range refusals, so the doc would still say only that a negative value is refused."
    fix: "In card 1's Intent, add that LoadConfig's doc comment names the refusal of a negative gate_hold_factor and of a gate_hold_min below 1."
---

### [MEDIUM] The done-run clear drops a run whose watch level never read

**Location:** `_lym/plan/03-run-watch-engine.md:84`

**Issue:** Step 1 skips a run with no level read yet, so it owes no line, and the card holds the clear only while a line is owed.
`clearDoneRuns` (`internal/dobbyengine/donerun.go:42-44`) waits for a landing-notice memory, and `unreadLandNoticeRuns` keeps an unread past-Land run remembered (`internal/dobbyengine/landnotice.go:100-108`).
The watch has no such hold, so a done run with an unread level loses its landed line, against the acceptance criterion that a landed run whose watch line has not posted is not cleared.

### [MEDIUM] The child re-read gate after a failed ReadChild is unspecified

**Location:** `_lym/plan/03-run-watch-engine.md:61`

**Issue:** The Memory paragraph defines `StatusSeen` and `ChildSeen` as the modification times "of the last child read", which leaves open whether a failed read advances them.
The analogous `childSeen` in `internal/dobbyengine/passes.go:349-350` is set unconditionally, so the natural implementation advances them.
A child halted at a stop then writes nothing more, and its stop line never goes.
The planned row checks only the failing tick.

### [MEDIUM] A stale land_notice copy can silence both landed lines

**Location:** `_lym/plan/03-run-watch-engine.md:59`

**Issue:** `rememberRuns` drops the landing memory of an unlisted run that owes nothing (`internal/dobbyengine/landnotice.go:159-163`), and a run first seen past Land is never remembered (`landnotice.go:117-118`).
`DobbyRuns` leaves out a run whose records do not read or whose code tip does not settle, so one such tick between the run's last tick ahead of Land and its landing ends with no landing notice.
The watch entry survives, as D4 requires, with its copy still true, so the landed line is skipped too.

### [MEDIUM] The level-unread skip also skips D3's copy

**Location:** `_lym/plan/03-run-watch-engine.md:58`

**Issue:** D3 copies the land_notice reading while the run is ahead of Land "once that reading exists", and D2's not-judged rule governs only the reading and its memory.
Step 1 says a run with no level read "is skipped whole", which takes step 2 with it.
A run whose `run_watch` first reads after Land then has a nil copy and is never skipped, so it posts a watch line beside the landing notice although land_notice read on.

### [MEDIUM] TestCheckLocalValues duplicates TestPrepareRunSettings

**Location:** `_lym/plan/01-run-watch-setting.md:14`

**Issue:** `configreg` wires `dobbyengine.CheckLocalValues` as the beam module's `CheckValues` (`internal/configreg/configreg.go:151`), and `runsettings.go:223-224` calls it.
`TestPrepareRunSettings` already holds the rows "a beam pull_request the lookup refuses", "a beam land_notice the lookup refuses" and "a beam autonomy the lookup refuses" (`internal/configreg/runsettings_test.go:82-104`).
The new key's refusal is one more row of that table, not a new test.

### [MEDIUM] The 20-process cap narrows D6 without a decision

**Location:** `_lym/plan/06-gate-hold-engine.md:56`

**Issue:** D6 says the notice names "every other process carrying the holder's marker", and the acceptance criterion names "each child PID".
Card 6 cuts both lists at 20 and adds a row for the cut.
A cap may be wise for a Go-side holder with a wide tree, but it is a design call the decision record does not make.

### [MEDIUM] Card 3 rewrites doc.go text under Extends

**Location:** `_lym/plan/03-run-watch-engine.md:22`

**Issue:** The rubric admits `Extends` only for adding to a ref without changing what is there.
The done-run-clear bullet in `internal/dobbyengine/doc.go:185-191` lists what a clear waits for, and card 3 adds a wait that bullet must name.

### [LOW] The board upsert help lists three per-task keys

**Location:** `_lym/plan/01-run-watch-setting.md:30`

**Issue:** `internal/boardcli/cli.go:586-587` documents `run_settings` as taking beam.pull_request, beam.land_notice and beam.autonomy, a closed list that run_watch extends.
The combined overlay with this fix passes `lym loom validate-plan --overlay` once `internal/boardcli/doc.go` sits under Uses.

### [LOW] docs/overview.md's list of dobby's mail goes stale

**Location:** `_lym/plan/04-run-watch-wiring.md:7`

**Issue:** `docs/overview.md:850` enumerates what dobby sends the nearest live orch, and the new lines are absent from it.

### [LOW] A nested run's orphans carry another marker

**Location:** `_lym/plan/05-gate-holder-marker.md:28`

**Issue:** D6 calls the marker "the entry every process it starts carries", but `runTest` replaces the entry in a nested run's environment with that run's own PID (`internal/gatecli/run.go:172`), and the nested run records no holder.

### [LOW] A failed state write loses the landed line

**Location:** `_lym/plan/03-run-watch-engine.md:85`

**Issue:** The watch step ends on an unwritable record before its owed line is stored, and the clear on the same tick (`internal/dobbyengine/loop.go:715-717`) reads a record that owes nothing.

### [LOW] The failing-LoadConfig row's reason is false

**Location:** `_lym/plan/06-gate-hold-engine.md:75`

**Issue:** In `tick`'s scheduling block the watch and gate-hold steps run before `startReady` (`internal/dobbyengine/loop.go:709-716`), so a failing `LoadConfig` reaches `gateHolds` through `tick` before `startReady` fails the tick.

### [NIT] LoadConfig's doc misses the new range refusals

**Location:** `_lym/plan/01-run-watch-setting.md:43`

**Issue:** `internal/dobbyengine/config.go:99-102` documents only the negative and per-task refusals, and the card's Intent updates that comment for run_watch alone.

## Rejected

- Card 6's "Keys of holds no longer held are dropped" read as pruning on a failed `TaskWorktree`: "no longer held" names a hold absent from `Holders`, which a failed `TaskWorktree` does not make, so the wording admits no duplicate notice.

## Focus notes

- Card 2 and card 3, episode stamp: `appendOrFold` either appends or raises the last entry's `Repeats` (`internal/shedengine/run.go:706-720`), and no history is trimmed, so history length plus last repeats rises with every new entry or fold; a switch of the stopped layer always changes batten's state too, so it posts. Not a defect.
- Card 5, marker process: `runTest` forms the marker with `os.Getpid()` (`internal/gatecli/run.go:172`) and the executor calls `acquireSlot` in the same process (`run.go:705`); a detached runner runs the gate tree in its own process. Not a defect, bar F10.
- Card 6 and card 7, holder matching: `acquireSlot` records `location.WorktreePath()` and `pairLocation` resolves `lymcwd.NamedWorktree(prime.HubPath, slug)`, both from lymcwd's resolved hub path; `dobbyengine` already imports `internal/proc` under its allowlist. Not a defect.
- Card 4 and card 7, `internal/beamcli/doc.go`: its seams section names no notice, so neither card falsifies it.
- The proposed target changes of F5, F6, F7, F8 and F9 pass `lym loom validate-plan --overlay` together against a clean `{}` baseline.
