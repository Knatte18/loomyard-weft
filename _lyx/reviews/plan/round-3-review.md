---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/05-hold-turn-end.md
    summary: Card 5 breaks TestAttach_AskingReentryStartsAtRecordedOffset in attach_asking_test.go, which is neither an Edit target of card 5 nor in its known-tests list; card 7 only rewrites it later
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/13-tail-in-stencils-and-steer.md
    summary: The driver-stencil replacements say that tail but the card never has the stencil state the tail's literal text, so the phrase has no referent for the driver agent
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/14-clock-seam.md
    summary: The exported Clock type is made by renaming the unexported clock under an Edit label, with no Rename typing and no plan handle, although it becomes public API that card 15 depends on
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/plan/14-clock-seam.md
    summary: Coverage paragraph claims every injected-clock test exercises SetClock, but attach_test.go and startup_test.go assign runner.clock directly and keep doing so
---

### [MEDIUM] Card 5 leaves a failing test at its own commit

**Location:** `_lyx/plan/05-hold-turn-end.md` (Edit list and "tests to rewrite" list); `internal/shuttleengine/attach_asking_test.go`

**Issue:**
Card 5 states a rule: rewrite every test that expects `OutcomeAsking` from `Wait`, `pollEventsTick` or an `Attach` that reaches `Wait`, "so the suite passes at this card's commit".
The rule is right, but its Edit list and its "known today" list miss one test.
`TestAttach_AskingReentryStartsAtRecordedOffset` in `attach_asking_test.go` seeds a legacy `asking` record with live strand, calls `runner.Attach`, and asserts `result.Outcome == OutcomeAsking` with the post-offset message, then asserts `run.json` carries an `AskingOffset` written by `finalize`.
After card 5, `Wait` holds instead of returning asking and `finalize` no longer writes `AskingOffset`, so the test runs to its one-minute deadline and fails on both assertions.
The file appears in no card-5 target list, and the later card that touches it (card 7) is not at or before card 5's commit.
The other asking-expecting files (`wait_test.go`, `waiting_test.go`, `attach_test.go`'s `BacklogEndsInAsk_ClassifiesAsking`, `completionsignal_enforcement_test.go`) are all listed.
Focus question 1: the card states a rule, not a name list, and the known-today list is otherwise complete, but this one file's test is the gap.

**Fix:**
Add `internal/shuttleengine/attach_asking_test.go` to card 5's Edit list and name `TestAttach_AskingReentryStartsAtRecordedOffset` in its known-tests list, adapted minimally at card 5 (the attached held run ends through the output file appearing, asserting Done, and the `AskingOffset` assertion goes).
Card 7 still rewrites that test into its final form, so say in card 7 that it rewrites the card-5 version.

### [LOW] Driver stencil says "that tail" without stating it

**Location:** `_lyx/plan/13-tail-in-stencils-and-steer.md`; `contracts/stencils/shed/shed-template-driver.md` `## Notifying the parent`

**Issue:**
Focus question 2: the card does reconcile all three sentences that conflict with the tail ("and nothing more", "send that notice verbatim instead of this generic line", "The notice is the whole message: add nothing to it"), so the section can state one rule.
The replacement wordings, however, are "nothing more than that tail", "followed by that tail" and "but that tail".
The card says the generic line and the notice "each end with that tail" but never says the stencil quotes the literal text; only the orch stencil bullet quotes "Answer briefly, then continue your task.".
The driver agent reads the stencil alone, so "that tail" has no antecedent and it cannot append the right words.

**Fix:**
Have the card add to the driver stencil's section one sentence quoting the tail literally ("Answer briefly, then continue your task."), with the three replacement sentences referring to it, and have the `claims_test.go` claim pin that literal in the driver stencil.

### [LOW] `clock` to `Clock` rename is typed as Edit and carries no handle

**Location:** `_lyx/plan/14-clock-seam.md`

**Issue:**
Focus question 3: the seam is a separately reviewable card, `Runner.SetClock` has its own `Create` handle, and the Intent states what a caller can skip and what bounds it (real clock by default, no production caller, a run takes the clock at `Start`/`Attach`, which `run.go` confirms); that part is sound.
The exported `Clock` type, however, is produced by renaming the unexported `clock` interface across `wait.go`, `run.go` and three test files, under an `Edit:` label with file-path targets.
The plan spec types a symbol rename as a `Rename` group (`old -> plan:` handle, AST-aware, with the plan's `## Rename mechanic` section), and `clock` is a common identifier (a field named `clock` of type `clock` in `Runner` and `Run`) where a text edit is error-prone.
`Clock` becomes public surface that card 15's test implements, yet no handle names it, so card 15's `Uses:` lists only `SetClock`.

**Fix:**
Type the rename as a `Rename` group in card 14 (`internal/shuttleengine#clock` -> `plan:internal/shuttleengine#Clock`), add the `## Rename mechanic` section to the overview, and list the `Clock` handle in card 15's `Uses:`.
Keep the Edit group for the remaining non-rename changes and `SetClock`.

### [NIT] Card 14's coverage sentence overstates

**Location:** `_lyx/plan/14-clock-seam.md`, Coverage paragraph

**Issue:**
"every injected-clock test in the package exercises the setter" is false: `attach_test.go` (`runner.clock = fc`, `mc`) and `startup_test.go` (`runner.clock = &multiStepClock{...}`) assign the field directly and the card leaves them so.

**Fix:**
Say that the tests built through `fixture_test.go`'s `withClock` exercise the setter.

## Focus notes

- Focus 4 (ImpactSummary of card 5): not a defect.
  It names its source ("its callers' packages") and pins no count of packages or assignments.
