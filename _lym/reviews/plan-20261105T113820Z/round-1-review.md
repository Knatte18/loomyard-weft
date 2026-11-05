---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: handler
    location: _lym/plan/13-board-promote-raddle.md:4 and _lym/plan/23-raddle-recipe.md:6
    summary: The recipe name raddle is declared twice, as boardengine.SketchRecipe (card 13) and shedrun.RecipeRaddle (card 23), breaking PATTERN-named-constants
    fix: In card 13's Create list, replace the line declaring `plan:internal/boardengine#SketchRecipe` with "- `plan:internal/shedrun#RecipeRaddle` -> `const RecipeRaddle = \"raddle\"`". In card 13's Intent, say that `boardengine` compares a task's recipe against `shedrun.RecipeRaddle` everywhere the card now names `SketchRecipe`; boardengine already depends on shedrun transitively, so the import adds no cycle. In card 23's Create list, delete the `plan:internal/shedrun#RecipeRaddle` line, and in its Intent, say that `RecipeNames` takes the constant card 13 declared.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/13-board-promote-raddle.md:37-38
    summary: A plain promote of a Sketch that still carries its label is refused by validateLabels with an upsert way forward, not the `lym board promote <slug> --recipe raddle` way forward that D2 and its acceptance criterion require
    fix: In card 13's Intent, have the refusal of a write that makes a discussed note with a stored record a task without recipe raddle name `lym board promote <slug> --recipe raddle` in every case, whether the note carries the sketch label or not. Either have `validateLabels` name that way forward when the entry was a discussed note before the write, or judge such a write with `checkDiscussedPromotion` before the label rule refuses it. In `TestDiscussedPromotion`, the plain-promote row of a Sketch carrying the label and the upsert-to-darn row of a Sketch then assert the `--recipe raddle` way forward.
  - id: F3
    severity: MEDIUM
    class: decision
    origin: lens:guard-reach
    location: _lym/plan/16-board-raddle-writes.md:28-33 and _lym/plan/10-raddle-proposal-spec.md:14
    summary: The project that the project and beam shapes file carries no type label, which validateLabels requires of a project, and no card says where the label comes from, so every project and beam filing is refused
    fix: Choose the project's type label and record it. One option is a `type` key in the spec's `project` object (card 10), checked by card 12's `type-invalid` and passed through card 22 into card 16's project `RaddleEntry`. Card 16's `TestFileRaddle` project and beam rows then assert the project's type label.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/12-raddleparser.md:38-45 and _lym/plan/22-raddle-file-producer.md:29
    summary: No D6 check refuses a part that builds a decision id, or carries a PATTERN id, that the record lacks, so such a proposal passes every check and then fails SplicePart at step 2 with no return or way forward
    fix: In card 12, add the checks `decision-unknown` (a part's decision id the record does not hold) and `pattern-unknown` (a part's PATTERN id the record's PATTERN changes do not hold), each with a row in `TestCheck`. In card 22, state that a `SplicePart` error at step 2 is `Stuck` with `Halt`, naming the error and the way forward.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/14-raddle-store-slug-rule.md:29
    summary: Card 14 says batten's start guard (card 24) catches an entry the replay landed under a held slug, but card 24 judges only raddle-recipe entries, so a darn task, tapestry task or project under a held slug meets no check
    fix: Reword card 14's Bound to say what holds. A board commit replayed from a machine that has not pulled the store can land a non-raddle entry under a held slug, and only a raddle entry is caught, by card 24's guard at its start. Add the consequence for `lym board raddle <slug>` and `lym report raddle`, which then name the old proposal. Alternatively, have card 15's replay apply the store-slug refusal to an added entry and set the add aside with a report, as it does for a store's lost entries.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/21-raddle-write-producer.md:31-38
    summary: Card 21 maps the raddle role into roleOpeningStencils, but its stencil content list omits the parent and edit directives that PATTERN-parent-directive and the opening-stencil directive tests require
    fix: In card 21's stencil content list, add that the stencil renders `{{.parent_directive}}` and `{{.edit_directive}}` as `contracts/stencils/loom/loom-template-discussion.md` does, so the opening-stencil directive tests in `contracts/stencils/parentdirective_test.go` pass.
  - id: F7
    severity: MEDIUM
    class: scope
    origin: lens:repo-invariants
    location: _lym/plan/23-raddle-recipe.md:46-47
    summary: arbiterRecipeDescription describes every recipe with no Finalize row as a task's Discussion, so a raddle run's arbiter is told the wrong thing about what its run does, and no card edits it
    fix: Add `internal/loomcli#arbiterRecipeDescription` to card 23's Edit list. In the Intent, describe a recipe with a `RaddleFile` row, chosen by its rows, as a run that splits a Sketch's stored record into board entries the operator approves, landing no code. The validate-plan overlay reports this glyph as glyph-not-found only because the validator runs from the weft anchor; the baseline reports the same for every existing code glyph, and the unit exists at internal/loomcli/arbiterprompt.go:27.
  - id: F8
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/plan/00-overview.md:64-68
    summary: The filing-level Shared Decision files entries by copying the Sketch's project field instead of using the Constraint's hubgeom parent read, and no Added-after-Discussion entry records this departure from the Constraint
    fix: Ask the parent with `lym creel post --parent --expect-reply` to record a decision (`lym loom decision add`) that filed entries take the Sketch's own `project` field, with `ParentBeam` reading their parent beam, in place of the PATTERN-pair-parent Constraint's hubgeom parent read. Until the decision is recorded, the filing-level Shared Decision cites it. If the parent declines, cards 16, 22 and 23 take the filing level through `hubgeom.ParentOf`.
  - id: F9
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/22-raddle-file-producer.md:30
    summary: D9 step 3 requires each filed task's body to name the Sketch slug and the part's contract and packages, but no card prescribes rendering that body or tests it
    fix: In card 22's step 3, state that each part's `RaddleEntry` body is rendered from the proposal and names the Sketch slug, the part's contract (takes, delivers, exports) and its packages with their sizes. Add an assertion of that body to `TestRaddleFile`'s tasks-shape row.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/23-raddle-recipe.md:86
    summary: Card 23 makes the claim at internal/loomcli/doc.go:25, that a finished run of a recipe with no Finalize row makes its entry a Sketch, false, and lists that file only under Uses
    fix: Constrained by uses-later-target, since cards 5, 6, 18 and 20 Use `internal/loomcli/doc.go`, so card 23 cannot simply target it. Drop `internal/loomcli/doc.go` from the Uses lists of cards 5, 6, 18 and 20, which change no behaviour that file describes. Then move it from card 23's Uses to its Extends, with an Intent sentence saying a recipe with a RaddleFile row ends its entry done instead of as a Sketch.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/13-board-promote-raddle.md:14
    summary: StatusDraft's doc comment says every new task takes the draft status, which card 13's released list and card 16's filing make false
    fix: Add `internal/boardengine#StatusDraft` to card 13's Edit list, with an Intent sentence that its comment names the released exception. The overlay's glyph-not-found for it is the same weft-anchor artifact as F7's; the unit exists at internal/boardengine/draft.go:6.
  - id: F12
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: _lym/plan/27-raddle-docs.md:9
    summary: The loom config key list in docs/overview.md gains neither raddle nor raddle_auto_approve, and no doc names the new report module or its cost_min_runs key
    fix: Extend card 27's Intent so that docs/overview.md's loom config key list names `raddle` and `raddle_auto_approve`, and its report module line names the `report` config module and `cost_min_runs`.
  - id: F13
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/13-board-promote-raddle.md:37
    summary: A task a plain promote made from a Sketch before this change still carries the sketch label with a recipe other than raddle, and after card 13 every validateWrite on it is refused; the card names no migration or bound for this state
    fix: In card 13's Intent, state how an existing task carrying `sketch` with a recipe other than raddle is handled. One option is that the label rule judges only a write that adds the label or changes the recipe, so an untouched legacy task keeps taking writes. Add a matching row to `TestValidateLabels`.
  - id: F14
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/13-board-promote-raddle.md:34
    summary: checkDiscussedPromotion runs only in boardCriticalSection, so a non-raddle promotion of a discussed note in another machine's commit replays onto origin unjudged, and card 13 states no bound for this bypass
    fix: Add a Bound to card 13's Intent, as card 14 has for its own rule. It says the replayer admits entries through validateWrite alone, so such a promotion can land on origin, and that card 24's start guard does not catch it because the task's recipe is not raddle.
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/24-batten-raddle-start-guard.md:26
    summary: The pass branch is justified by a resumed raddle run, but a resume has a status file, and refuse runs the check only for a run with no status file
    fix: Reword card 24's rationale for the own-stamp pass. The state that reaches it is a fresh `lym batten run` of the raddle entry after a cancel that kept the entry and removed its status, past a filing that stored the Sketch's directory.
  - id: F16
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/25-pawl-raddle-handoff.md:39
    summary: Deps.Notify drops NotifyOrchAt's posted result, so a hand-off that posts nothing still writes the marker and is lost for good
    fix: In card 25, have the hand-off step treat a post that reached no orch (posted false) as a failure. It halts as `Stuck` with `Halt`, with no marker written, so a resume retries. Name a TestBeamUp row for it.
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/12-raddleparser.md:45,50
    summary: The Sketch is the run's own open entry, so a part slug equal to it always also raises slug-taken, and TestCheck's row asserting slug-sketch alone cannot hold
    fix: In card 12, have `slug-taken` exempt the run's own entry, which `slug-sketch` judges, so the slug-sketch row breaks that rule alone.
  - id: F18
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/04-usage-record-writer.md:72-76
    summary: Card 4 leaves facts to be found while it runs, namely whether each background file already states the moved rule and which takers hold the teardown lock shared, though both are known now
    fix: In card 4, record the researched facts and drop the conditionals. pattern/PATTERN-remote-refs-lock.md already states all three moved rules (lines 5, 10 and 11), so nothing moves there. pattern/PATTERN-shed-recipe-registry.md states the key lookup, the Command row and the entry_gate rule (lines 8, 13, 22 and 27), so only the rule about an off that reads more its own call moves. pattern/PATTERN-test-speed.md states the `//lym:seam`, injected-clock and `t.Parallel` rules (lines 94-112), so only the rule that a skip never names the OS alone moves. The teardown lock is held shared only by the push paths (internal/fabricengine/teardownlock.go:81, internal/fabricengine/pushanchored.go:164) and exclusive at teardownlock.go:114 and remoteteardown.go:316 and 364, so the shortened slug-remote-order line holds.
  - id: F19
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/03-shuttle-session-end-hook.md:31
    summary: The first row of TestRunner_SessionEnd, a runner with no hook running a done run as before, repeats what every existing finalize test already covers with a nil hook
    fix: Drop that row from TestRunner_SessionEnd. TestRun_Finalize_ReportsUsageAndTimes and the other run tests already drive done runs on runners with no hook set.
  - id: F20
    severity: NIT
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/23-raddle-recipe.md:94
    summary: AmendmentTarget's doc comment lists the recipe names, as do the RowNames, Rows and RecipeEngines comments that card 23 updates, but card 23 leaves it out
    fix: Add `internal/loomrecipe#AmendmentTarget` to card 23's Edit list, and name its doc comment beside the RowNames, Rows and RecipeEngines comments that gain raddle. The overlay's glyph-not-found for it is the weft-anchor artifact; the unit exists at internal/loomrecipe/names.go:102.
---

### [BLOCKING] The recipe name raddle is declared twice

**Location:** `_lym/plan/13-board-promote-raddle.md:4`, `_lym/plan/23-raddle-recipe.md:6`

**Issue:**
- Card 13 creates `boardengine.SketchRecipe = "raddle"` and card 23 creates `shedrun.RecipeRaddle = "raddle"`. That is the same recipe name declared twice, which `PATTERN-named-constants` forbids: one named constant, declared once and referenced everywhere.
- `go list -deps ./internal/boardengine` already includes `internal/shedrun`, so boardengine can reference the shedrun constant with no import cycle.
- The proposed fix was overlaid with `lym loom validate-plan --overlay` and added no blocking finding against the `{}` baseline.

### [MEDIUM] D2's way forward is not named for a plain promote of a labelled Sketch

**Location:** `_lym/plan/13-board-promote-raddle.md:37-38`

**Issue:**
- D2 says that any write making a discussed note with a stored record a task, other than with recipe raddle, is refused naming `lym board promote <slug> --recipe raddle`. The D2 acceptance criterion repeats this "whether the note carries the label or not".
- Card 13 lets `validateLabels` refuse the most common case first: a plain promote of a Sketch that still carries its label. That refusal names an upsert of the recipe or dropping the label instead.

### [MEDIUM] The project filed by the project and beam shapes has no type label

**Location:** `_lym/plan/16-board-raddle-writes.md:28-33`, `_lym/plan/10-raddle-proposal-spec.md:14`

**Issue:**
- `validateLabels` refuses a project with no type label (`internal/boardengine/labels.go:39-40`).
- Card 16 files the project without the `sketch` label, and the spec's `project` object holds only a slug and a title, so nothing supplies the label.
- I found this on my own as well as through the guard-reach lens.

### [MEDIUM] No check catches a decision id or PATTERN id the record lacks before step 2's splice

**Location:** `_lym/plan/12-raddleparser.md:38-45`, `_lym/plan/22-raddle-file-producer.md:29`

**Issue:**
- `SplicePart` errors on an unknown id (card 11), but no D6 check refuses a part's unknown decision id or PATTERN id.
- Such a proposal passes the gate, `raddle-approve` and Raddle-File's check, and then fails at step 2.
- Card 22 gives that failure no return and no halt with a way forward.

### [MEDIUM] Card 14's Bound overstates what card 24's guard catches

**Location:** `_lym/plan/14-raddle-store-slug-rule.md:29`

**Issue:**
- Card 24's guard runs only when the entry's recipe is raddle.
- A non-raddle entry that the replay lands under a held slug meets no check, which breaks D10's claim that a store slug always names the entry that filed or was raddled under it.

### [MEDIUM] The raddle stencil omits the parent and edit directives

**Location:** `_lym/plan/21-raddle-write-producer.md:31-38`

**Issue:**
- `PATTERN-parent-directive` requires a spawned role's stencil to render the parent and edit directives. `contracts/stencils/parentdirective_test.go` (lines 183-205) asserts this for every opening stencil.
- Card 21 registers the raddle stencil as an opening stencil, but its content list names neither directive.

### [MEDIUM] arbiterRecipeDescription misdescribes a raddle run

**Location:** `_lym/plan/23-raddle-recipe.md:46-47`

**Issue:**
- `internal/loomcli/arbiterprompt.go:40-41` describes every recipe with no Finalize row as "a task's Discussion: its decision record is written, reviewed and stored". A raddle run would get that description, though it files board entries from a stored record.
- No card edits the function.

### [MEDIUM] The filing-level Shared Decision departs from the PATTERN-pair-parent Constraint unrecorded

**Location:** `_lym/plan/00-overview.md:64-68`

**Issue:**
- The decision record's Constraint says: "filed entries take the Sketch's parent level through `hubgeom`'s one parent read".
- The plan copies the Sketch's `project` field instead. Its reasoning is sound: `ParentOf` answers the nearest beam and skips a grouping project (`internal/hubgeom/parentof.go:14-31`).
- But the Constraint conflicts with D9's "at the Sketch's own project level", and no Added-after-Discussion entry settles which one binds. The plan may not settle that departure from a Constraint on its own.

### [MEDIUM] Nothing renders the filed task's body that D9 requires

**Location:** `_lym/plan/22-raddle-file-producer.md:30`

**Issue:**
- D9 step 3 requires filed tasks to carry "a body naming the Sketch slug, the part's contract and packages".
- Card 16 has a body field, but card 22 builds the filing without saying what goes in the body, and no test asserts it.

### [LOW] internal/loomcli/doc.go is left stale by card 23

**Location:** `_lym/plan/23-raddle-recipe.md:86`

**Issue:**
- `internal/loomcli/doc.go:25` says a run with no Finalize row, a motif run, makes its entry a Sketch. Card 23 makes that false for raddle.
- Overlaying the simple fix, which moves the file into card 23's targets, raised `uses-later-target` for cards 5, 6, 18 and 20. The fix proposed instead first drops those cards' Uses of the file.

### [LOW] StatusDraft's doc comment becomes false

**Location:** `_lym/plan/13-board-promote-raddle.md:14`

**Issue:** The comment at `internal/boardengine/draft.go:5` says "every new task takes it", but card 13's released list and card 16's filing release new tasks.

### [LOW] The overview doc misses the new config keys

**Location:** `_lym/plan/27-raddle-docs.md:9`

**Issue:** The new loom keys (card 9) and the new report config module (card 7) are absent from `docs/overview.md`'s key lists, and card 27 updates only the recipe lists and verbs there.

### [LOW] Legacy tasks carrying the sketch label become unwritable

**Location:** `_lym/plan/13-board-promote-raddle.md:37`

**Issue:**
- Today `Store.Promote` copies a Sketch's labels to the task (`internal/boardengine/store.go:771-774`).
- After card 13's label rule, every write to such an existing task fails validation, and the card states no handling or bound.

### [LOW] The discussed-promotion rule has an unstated replay bypass

**Location:** `_lym/plan/13-board-promote-raddle.md:34`

**Issue:** The new rule runs only in `boardCriticalSection`. A promotion replayed from another machine skips it, and card 13 states no bound for this, as the rubric's attack-surface rule requires.

### [LOW] Card 24's pass-branch rationale names an unreachable state

**Location:** `_lym/plan/24-batten-raddle-start-guard.md:26`

**Issue:** `refuse` judges the raddle check only for a run with no status file (`internal/battencli/startguard.go:168,195`). A resumed run has a status file, so a resume never reaches this branch.

### [LOW] A hand-off that reaches no orch is lost for good

**Location:** `_lym/plan/25-pawl-raddle-handoff.md:39`

**Issue:**
- `NotifyOrchAt` returns posted false with nil error when no orch strand is recorded (`internal/orchcli/notify.go:68`).
- Card 25 writes the marker after any nil-error notify, so a silent miss is never retried.

### [LOW] slug-sketch never fires alone

**Location:** `_lym/plan/12-raddleparser.md:45,50`

**Issue:** The Sketch is the run's own open entry until step 7, so a part slug equal to it also trips `slug-taken`. That contradicts TestCheck's "breaking that rule alone" rows.

### [LOW] Card 4 leaves facts to be found at implementation time

**Location:** `_lym/plan/04-usage-record-writer.md:72-76`

**Issue:**
- Card 4 tells the implementer to grep the teardown lock's takers, and to move rule text "when the file does not already state it".
- Both facts are knowable now, which makes them research steps. The fix records them.

### [LOW] TestRunner_SessionEnd repeats a nil-hook row

**Location:** `_lym/plan/03-shuttle-session-end-hook.md:31`

**Issue:** The row "a runner with no hook set runs a done run as before" repeats behaviour that `TestRun_Finalize_ReportsUsageAndTimes` and the other run tests already cover with a nil hook.

### [NIT] AmendmentTarget's recipe-name comment is missed

**Location:** `_lym/plan/23-raddle-recipe.md:94`

**Issue:** `internal/loomrecipe/names.go:101` lists the recipe names in `AmendmentTarget`'s comment, the same list card 23 updates in its sibling functions' comments.

## Rejected

- The affected-set lens's F1 (refusal texts in `store.go:373/377`, `task.go:300/307` and `adopt.go:601` advise a plain `lym board promote`) is rejected. Those texts address notes in general, such as a note with `depends_on`. For a Sketch, the plain promote then meets card 13's refusal, which with F2 fixed names `--recipe raddle`, so the user is led to the right command in two steps and no text is false.

## Validator note

`lym loom validate-plan --overlay` runs only from the weft anchor, where the code tree's files and units are absent. Its baseline therefore already reports path-missing and glyph-not-found for every existing code target, including `validateLabels` and `finishesAsSketch`. The fixes were judged by the diff against that baseline:

- F1's fix overlaid alone added no blocking finding.
- The combined overlay of F1, F7, F10's simple form, F11 and F20 added glyph-not-found only for the new code glyphs. That is the same artifact; each unit was confirmed in the code tree.
- It also added `uses-later-target` for the `internal/loomcli/doc.go` move. That is why F10's fix first drops those Uses.
