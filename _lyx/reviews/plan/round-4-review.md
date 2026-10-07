---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/07-two-agent-round.md:114
    summary: Run covers only an ErrNotStarted start failure, so a hard StartGated error on the fixer (for example a pre-existing fixer-report path) leaves the started reviewer live, and no test row pins it
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/07-two-agent-round.md:122
    summary: Card 7 has run.go read the marker root and base off the wired burlerCLI, but the struct lives in internal/burlercli/cli.go, which is not among card 7's targets
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/02-ready-marker-path.md:33
    summary: The pathRules row builds its review path from l.AnchorPath(), which is relative in that table, so Path would resolve it against root a second time and the row would not exercise the _lyx drop
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/07-two-agent-round.md:160
    summary: ImpactSummary of cards 7 and 1 claims only named files are touched, which their own target lists contradict, and card 7's summary says nothing about the deleted stencil's blast radius
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/08-two-half-resume.md:77
    summary: Card 8's Resume test reproduces the skipped-handoff, half-died and changed-review rows that card 7's failure-rules test already covers through the shared join
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/05-round-stencils.md:44
    summary: The shared pattern and friction directive stencils still describe a combined review-and-fix agent with part A and part B, and no card targets them
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/07-two-agent-round.md:111
    summary: Card 7 states two halts are outside the refusal table but is silent on the marker removal and marker write failures, which also halt the round through the loom failure path
---

### [MEDIUM] Run names only the ErrNotStarted start failure

**Location:** `_lyx/plan/07-two-agent-round.md:114-117`, and the test paragraph at `:148`.
**Issue:** The `Run` paragraph defines what happens when a start fails with `ErrNotStarted`: the half reports `OutcomeDied` with `NotStarted`, a failed reviewer leaves the fixer unspawned, and a failed fixer stops the reviewer.
`Runner.StartGated` has a second failure class: every pre-strand failure (spec validation, run-directory creation, `AddStrand`, run-state persistence) returns a plain error that does not wrap `ErrNotStarted`.
That class is reachable here.
`shuttleengine.Spec` validation rejects an `OutputFiles` entry that already exists, and `Profile.validate` does not check the fixer-report path, so a stale fixer-report beside a fresh review path passes `Run`'s validation, starts the reviewer, and then fails the fixer's `StartGated` with the plain error.
The card does not say that this error stops the reviewer, so a literal implementation returns the error with the reviewer's strand still live.
That is the round-ends-with-a-live-half state D6 and its acceptance criterion rule out.
The test list pins a reviewer that never started and a fixer that never started, both as `ErrNotStarted` results, and no row for the plain error.
**Fix:** In the `Run` paragraph, state that any error from the fixer's `StartGated`, `ErrNotStarted` or not, stops the already-started reviewer through the remover before returning, and that a plain error from the reviewer's `StartGated` returns it with the fixer never started.
State which value the round returns for the plain error: the start error, wrapped as `ErrHalfNotStopped` instead when the stop fails.
Add a row to the failure-rules test for a fixer whose start fails with a plain error, asserting the reviewer's guid was removed and the error is returned, and a row for a reviewer whose start fails with a plain error, asserting the fixer was never started.

### [MEDIUM] `internal/burlercli/cli.go` is missing from card 7's targets

**Location:** `_lyx/plan/07-two-agent-round.md:122` (the `burlercli/run.go` paragraph) against the target list at `:12-28`.
**Issue:** `run.go` must call `burlermarker.Path` with root and base "taken from the wired geometry" (hub: the anchor for both; standalone: the target and the state directory).
`burlerCLI` carries `engine`, `cwd`, `mode`, `stateDir` and `stencilsDir`, and no anchor or target.
`burlerengine.Engine` keeps its `Geometry` unexported, and the engine derives no path, so `run.go` cannot read it back.
The root and base therefore have to be new fields of the `burlerCLI` struct, which is declared in `internal/burlercli/cli.go`.
Card 7 lists `run.go`, `wiring.go` and the two test files for `burlercli` but not `cli.go`, and card 4 is the only card that edits `cli.go`, for the verb registration.
A card executor limited to the listed targets cannot add the field.
**Fix:** Add `internal/burlercli/cli.go` to card 7's `**Edit:**` list and say in the `burlercli` paragraph that `burlerCLI` gains the marker root and base fields, set by `wireHub` and `wireStandalone`.

### [LOW] The pathRules row for `burlermarker.Path` feeds it a relative review path built from the anchor

**Location:** `_lyx/plan/02-ready-marker-path.md:33`.
**Issue:** The card says the `classTransient` row passes "a review path under `l.AnchorPath()`'s `_lyx`" with the anchor as both root and base.
In `cmd/lyx/pathrules_test.go` the fixture hub is `filepath.Join("home", "user", "repo-LYXHUB")`, so `l.AnchorPath()` is a relative path in both fixtures.
`Path` resolves a relative review path against `root`, so a review path that already starts with the anchor is joined under the anchor a second time.
The marker then lands under `<anchor>/.lyx/<anchor>/_lyx/...`, which still satisfies the table's under-`.lyx`, not-under-`_lyx` and anchor-based-root assertions, so the row passes without exercising the leading-`_lyx` drop it exists to guard.
**Fix:** Name the row's inputs: the review path is relative to the anchor (`filepath.Join(lyxdirs.LyxDirName, "reviews", ...)`), and the anchor is both root and base, so both fixtures walk the rule the card states.

### [LOW] ImpactSummary exclusivity claims contradict the target lists

**Location:** `_lyx/plan/01-shuttle-probe.md:35` and `_lyx/plan/07-two-agent-round.md:160`.
**Issue:** Card 1 says "the budget row is the only other file touched", while its `**Edit:**` list also holds `attach_test.go` and `doc.go`.
Card 7 says the refusal-spec row, the stencil doc and the budget row "are the only other files touched", while its targets also include `docs/overview.md`, the `burlerengine` and `shedadapters` `doc.go` files, `contracts/stencils/stencils.go`, `claims_test.go`, `.gitattributes` and `pattern/PATTERN-review-round.md`.
Card 7 also carries a `**Delete:**` target, and its ImpactSummary does not say who referenced the deleted orchestrator stencil (its embed, registry entry, `roleOpeningStencils` entry, claims, `.gitattributes` line, `prompt.go`, `doc.go`) or that all of them are in the card.
**Fix:** Reword both summaries so they claim no exclusivity the targets contradict, and give card 7's summary the Delete blast radius: every reference to the deleted stencil is a target of this card.

### [LOW] Card 8's Resume test repeats card 7's failure rows

**Location:** `_lyx/plan/08-two-half-resume.md:77`.
**Issue:** The new `Resume` function "reproduces the skipped-handoff, half-died and changed-review rows".
Card 7's failure-rules function already covers every failure row of `join`, and its marker-lifecycle function covers the marker write conditions; `Resume` hands its attached handles to the same `join`, and the same `reviewReady`.
`PATTERN-test-economy` makes a case already covered a row of the existing test, and a vague overlap is not what it asks for here: the covering tests are card 7's failure-rules and marker-lifecycle functions.
**Fix:** Keep in the `Resume` function only what is resume-specific (both handles attached with neither wait blocking the other, a stale marker removed before attaching, nothing spawned, the marker written after the attached reviewer is done), plus one failure row that proves the attached handles route into `join`; the three failure outcomes stay pinned once, through card 7's table.

### [LOW] The directive stencils still describe a combined agent

**Location:** `_lyx/plan/05-round-stencils.md:44-50` (targets) and `_lyx/plan/07-two-agent-round.md:90-95`.
**Issue:** `contracts/stencils/pattern/pattern-directive-review-fix.md` is given to both halves (D12) and its header says it is for the "combined review+fix agent that judges and then changes code", with a body keyed to "part A" and "part B".
`contracts/stencils/friction/friction-directive-review-fix.md` goes to the fixer alone (D4), and its header calls it the directive "for the Burler round's combined review-then-fix agent"; its body asks about friction "in the review phase or the fix phase".
`clusterRulesBlock` in `prompt.go` tells the reviewer the review must be written "before job B touches anything", and `toolUseRules` is phrased in terms of job A.
No card targets the two directive stencils, and card 7's `prompt.go` paragraph rewords neither block.
After the split the header comments are false, and the part A and part B wording is not defined by either new orchestrator.
**Fix:** Add both directive stencils to card 5's `**Edit:**` list: their header comments name the two halves, the pattern directive says which half each part binds, and the friction body speaks to the fixer's own phases.
Have card 7's `prompt.go` paragraph reword `clusterRulesBlock` and `toolUseRules` for the reviewer-only and shared cases, and extend the existing claims rows if the wording is pinned.

### [LOW] Marker I/O halts are not placed relative to the refusal table

**Location:** `_lyx/plan/07-two-agent-round.md:111-112`.
**Issue:** The card states that the skipped-handoff and changed-review errors are burler-module errors outside `contracts/specs/refusal-spec.md`'s table.
`Run` also removes a stale marker before starting, and `reviewReady` creates the marker's parent directory and writes it.
A failure of either returns an error through the same failure path and halts the round, and the card does not say that it carries a way forward or is outside the table.
**Fix:** Add the marker removal and write failures to the sentence that places the skipped-handoff and changed-review errors outside the table, and name the marker path in their messages.

## Focus outcomes

- **Path constructors and `pathrules_test.go`.** `burlermarker.Path` is the only new exported never-tracked path constructor, it has its `classTransient` row, and the Shared Decisions bullet triggers on "a path constructor". No constructor lacks a row; the row's inputs are F3.
- **Card 7's `docs/overview.md` coverage.** The Docs paragraph names the burler entry (which says "one agent") and the execution-stack walkthrough (which says "run one handler agent" and a single Stop hook); the stack-table row and the references blurb say "A-review → B-fix", which stays true. Not a defect.
- **Card 7's start failures.** Pinned for `ErrNotStarted`, not for a plain start error: F1.
- **Card 7's halts.** `ErrHalfNotStopped` carries a way forward and rides the widened `producer re-run: superseded strand not removed` row; the skipped-handoff and changed-review errors are stated outside the table; the marker I/O halts are F7.
- **ImpactSummary.** Each summary is a blast-radius conclusion and none restates its Intent; the exclusivity claims of cards 1 and 7 are F4.
