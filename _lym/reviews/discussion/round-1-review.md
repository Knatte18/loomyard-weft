---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md (D6, run-lock admission paragraph)
    summary: D6 admits the conversion by a shape that fixes only kind, status, discussed and the body prefix, so any writer may end the run's hold and change the title, labels, depends_on or the rest of the body, and D6 states no bound
    fix: In D6, replace the shape sentence with an admission equal to a derived project of the entry before the write, as endsAsSketch does for a Sketch. The admitted change is kind project, no status, and the fields a run, a landing or the dependency graph reads cleared, named as the set boardengine's sketchOf clears (depends_on, isolated, recipe, profile, run_settings, pull_request, notify_on_land). Title, module labels, project and beam stay unchanged, the body is exactly the raddle project prefix for the slug, the part list and the former body, and order is exactly the filed parts. Add a Bound line saying the admission lets through only that change and refuses any other scope change alongside it, and add an acceptance criterion that a write of the converted shape with a changed title or depends_on is refused by the run lock.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md (D6, post-filing status skip)
    summary: The status-write skip is keyed on the run's raddle store, but the only raddle store is the board's per-slug directory, which no clearing of the run's records removes, so the skip never ends and a later beam run under the slug loses its status writes
    fix: In D6, key the skip on a run-scoped record that the arras extension of battencli.ClearMotifRun removes, and name it (for example a mark in the run's raddle state under .lym, written when Raddle-File's board write lands with a project). Say the board raddle store under raddle/<slug>/ is not the key. Extend the D6 criterion so that after lym beam create on the converted project, the beam run's status writes reach the entry.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md (D2 loom keys bullet, D4 Bound)
    summary: D2 gives the split perch an auto-accept setting, which turns a spent budget into an accept of a CONTINUE or CIRCLING round, and in perch mode that accept files with no operator approval; neither D2 nor D4's Bound names this route or bounds it
    fix: In D2's loom keys bullet, state the split perch's auto-accept key, its default false, and that it is a guard-category key, as reviewbudget registers each perch's auto-accept key. In D4's Bound, add that with that key on, a spent budget files a proposal whose last round still had open findings, and that the bound is the key's operator-only guard category, every proposal check in Raddle-File and the filed tasks staying draft. Add an acceptance criterion that with the key off a spent budget does not reach Raddle-File.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md (D10)
    summary: D10's rationale is false in the family outcome, since a dependency on a plain project already waits until every landing member is done and draft members are landing members, so re-pointing dependents there adds board writes and a filing-failure path for no gain
    fix: Limit D10 to the one-task outcome. State in D10 that in the family outcome dependents keep depends_on on the converted project, because boardengine.Dependencies holds a dependency on a plain project until every landing member is done. Change the D10 acceptance criterion to - an entry depending on an arras entry depends, after a one-task filing, on the filed task, and after a family filing still on the project, and does not start before every member is done.
  - id: F5
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md (D8 Bound, first bullet)
    summary: D8 names lym board set-status as the only trigger, but that verb calls the shared Board.SetStatus, which other writers such as batten cancel's claim clear call with a null status, and the record does not place the release where only the verb reaches it
    fix: In D8's first Bound bullet, add that the member release lives in the verb's own path or a board method only the verb calls, never in Board.SetStatus or Store.SetStatus, so the other null-status writers (batten cancel's claim clear among them) release no member. Extend the D8 criterion that an upsert or merge releases no member to name batten cancel's claim clear on a converted project.
  - id: F6
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md (D2, D4)
    summary: Neither D2 nor D4 says how Raddle-Write ends and stores the cold proposal in perch mode, nor which return handling a re-armed cycle uses; today's code splits auto from every other mode, so perch would take the interactive branch that raddle-approve now refuses
    fix: In D4, add that in perch mode Raddle-Write ends on the finished proposal and the raddle gate's pass stores the cold proposal, as in auto mode, and that a return from Raddle-File moves the last proposal aside as in auto mode. Add an acceptance criterion that an arras run's Raddle-File finds a stored cold proposal after the perch's approval, also after a return.
  - id: F7
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md (D7 Bound, way forward after the filing)
    summary: After the filing the refusal names the filed task's own run, but a filed part is a draft with no run, and in the family outcome there are several parts, so the way forward leads to the existing refusal to start a run
    fix: In D7's Bound, replace the after-filing way forward with one that reaches a recorder - the decision is recorded with lym loom decision add on the part it concerns once the operator has released it and its run has passed its record row, and the refusal lists the filed slugs read from the run's filing. Add the case to the D7 refusal criterion.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md (D7 Bound, Scope Out first bullet)
    summary: D7's refusal applies to any run whose recipe holds a RaddleFile row, which includes the raddle recipe, while Scope Out says raddle keeps its behaviour
    fix: Either key D7's refusal on perch mode, so plain raddle runs keep today's behaviour, or name in Scope In that the refusal from Raddle-File on also applies to raddle runs and why. Keep the decision by rows, never by recipe name.
  - id: F9
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md (D10, refusal sentence)
    summary: A board refusal of the dependents' re-point returns the run to Raddle-Write, but no re-split clears a cause on another entry, so each cycle fails the same way until the third-failure halt
    fix: In D10, make a board refusal of the re-point a halt for the run's parent that names the refusing entry and the refusal as its way forward, not a return to Raddle-Write. Add a criterion that a refused re-point halts naming the entry.
  - id: F10
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md (Open risks, third bullet)
    summary: The Open risks line asking the parent whether a cold fixer meets raddle fixes what it finds is a mechanism choice the writer can make, not a question only the parent can settle, and D2 already states and grounds the choice
    fix: Drop the third Open risks bullet. In support-log.md's Question ledger, change the matching Open line to Resolved (auto-pick) with its reason, the raddle stage fixing its proposal through a cold fixer on the raddle model.
---

### [MEDIUM] D6's run-lock admission by shape is unbounded

**Location:** `_lym/discussion/decision-record.md`, D6, the paragraph starting "The run lock admits the change by its shape".
**Issue:** The admitted shape fixes kind, status, the discussed flag and the body's opening only. `checkRunLocks` passes a Sketch change only when the new entry equals `sketchOf(before)` in every scope field (`internal/boardengine/sketch.go:74-79`, `internal/boardengine/runlock.go:164`). D6's admission has no such equality. A `lym board upsert` from the run's own agent that writes the shape, with a new title, labels, `depends_on` or arbitrary body text after the prefix, would pass and end the run's hold. The attack-surface rule requires the Decision to state what the admission lets through and what bounds it. This also settles focus item 2: the fields the entry loses are the ones `sketchOf` clears, which F1's fix names.

### [MEDIUM] D6's status-write skip has no end

**Location:** `_lym/discussion/decision-record.md`, D6, "The skip is keyed on the run".
**Issue:** The raddle store is the board's `raddle/<sketch>/` directory (`internal/boardengine/raddlestore.go:84`), and `RaddleStore.Holds` matches on `entry.Sketch == slug` (`raddlestore.go:45-48`). Nothing outside `boardengine` clears it, and `clearMotifRun` clears only run records (`internal/battencli/cleanup.go`). Read as written, the skip never ends. A later beam run under the slug would then lose its status writes, which is the harm the support log gives for rejecting the body-keyed skip.

### [MEDIUM] The split perch's auto-accept key bypasses the only approval in perch mode

**Location:** `_lym/discussion/decision-record.md`, D2's loom keys bullet and D4's Bound.
**Issue:** `internal/shedadapters/bouncer.go:249` and `bouncer.go:1352`: with AutoAccept set, a spent budget accepts a CONTINUE or CIRCLING round. D4 removes the operator approval on the grounds that the perch approves. D2 adds a key that lets the perch approve with findings still open, and states neither its default nor its guard category.

### [MEDIUM] D10's premise is false for the family outcome

**Location:** `_lym/discussion/decision-record.md`, D10.
**Issue:** `internal/boardengine/dependency.go:10`: "A plain project is done when its status is done, or when it has a landing member and every landing member is done", and members are the tasks whose `project` names it (`dependency.go:22`). A dependent left on the converted project waits for the whole family. The rationale holds only for the one-task outcome, where the entry ends done. This settles focus item 3. Keeping dependents on the project is also what D6's stated purpose favours, since the board shows the family as the arras entry's result.

### [LOW] D8's trigger is not placed out of reach of other null-status writers

**Location:** `_lym/discussion/decision-record.md`, D8 Bound.
**Issue:** The verb calls `Board.SetStatus` (`internal/boardengine/board.go:212`), and the lens traced other callers that write a null status through it. The Bound names the trigger but not where the release sits, so a plan that places it in `Board.SetStatus` would let a cancel's claim clear release a family.

### [LOW] Perch mode's end of Raddle-Write is unstated

**Location:** `_lym/discussion/decision-record.md`, D2 and D4.
**Issue:** `internal/loomshed/raddlewrite.go` stores the cold proposal on the gate's pass only in auto mode and moves a returned proposal aside only in auto mode. With `raddle-approve` refused in perch mode, nothing stores the cold proposal that Raddle-File stages.

### [LOW] D7's after-filing way forward reaches no recorder

**Location:** `_lym/discussion/decision-record.md`, D7 Bound.
**Issue:** Filed parts are drafts (`internal/boardengine/raddlefile.go:82`), so `lym loom decision add` on one hits the start-the-run refusal (`internal/loomcli/decision.go:71`). In the family outcome the record also names no part. The refusal does not lead the operator to a working step.

### [LOW] D7's refusal also changes raddle runs

**Location:** `_lym/discussion/decision-record.md`, D7 Bound and Scope Out.
**Issue:** `contracts/recipes/raddle-recipe.yaml` holds a `RaddleFile` row, so the refusal applies there too. Scope Out says raddle keeps its behaviour.

### [LOW] D10's refusal returns to a re-split that cannot clear it

**Location:** `_lym/discussion/decision-record.md`, D10.
**Issue:** A run lock on another entry does not change with a new proposal. Each return spends a full perch cycle, and the run ends at Raddle-File's third-failure halt with no named cause.

### [LOW] An Open risks line asks the parent a writer's choice

**Location:** `_lym/discussion/decision-record.md`, Open risks, third bullet.
**Issue:** The rubric's `decision` class covers an outdated entry, a conflict with another run or a landed commit, and doubt about what the task is for. Which agent fixes the proposal is a mechanism choice inside the entry's stated idea. D2 makes it and the support log's Rejected alternatives ground it. Raising it to the parent adds a stop the arbiter cannot settle for a question the writer has already answered. This settles focus item 6. The other two lines stay: the first names a sequencing dependency on another run's landed text, and the second states an accepted gap that D7 bounds.

## Verdicts on focus items with no finding

- Focus 1, D6 and the rubric's own-entry bullet: D6 does not ask the run's agent to edit its own entry through `lym board`. The filing write is Go's Raddle-File, and the admission is a change to the board module's run-lock code, which the bullet exempts, as the motif Sketch change shows. The bound on the admission is F1.
- Focus 4, D7 at Raddle-Write after a return: `lym loom decision add` judges the current row, so after a return to Raddle-Write it records again. D2's re-armed perch reads the record every round, so it reviews a decision added then. The refusal at a halted Raddle-File points at `lym loom goto --to Raddle-Write`, which passes through the Raddle-Write session that re-arms the perch. Held.
- Focus 5, D4 against a hand-edited recipe: only recipes embedded in the binary name settings (`internal/shedrecipe/doc.go`), and the mode is recorded once from the embedded rows, so a hub cannot add a split perch to another recipe. Held. The auto-accept route is F3, and the new release path's placement is F5.
- Focus 7, leanness: the Auto-mode assumptions are one-line pointers by id, which autonomous mode requires. No Decision paragraph carries deliberation the plan writer does not need. No relocation finding.

## Rejected

- The walkthrough lens noted that today's `filing` builds the project entry from the proposal's title, type label and a prefix-only body, unlike D6's end state. D6 states the end state, so changing the builder is plan work, not a record defect. The lens itself did not raise it as a finding.
