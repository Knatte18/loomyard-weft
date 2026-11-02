---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: pattern/PATTERN-parent-directive.md:5
    origin: lens:code-affected-set
    summary: PATTERN-parent-directive still names the discussion role as the one place a question reaches a person, but Drift-Verify now fills mode_rules with operator text too
    fix: In pattern/PATTERN-parent-directive.md bullet 1, state that an interactive role's questions reach a person only through the mode_rules marker filled by Go, and name the Discussion chair and the Drift-Verify seat under drift.yaml verify_interactive as the roles that do. Keep the rule that no stencil file tells an agent to ask the operator.
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/driftcli/wiring.go:69
    origin: lens:code-tests
    summary: The read of OperatorSessionOpen into VerbStatus.OperatorSession in verbStatus has no test, so a dropped field would record operator acceptances as agent and unruled with every driftengine test still green
    fix: Add one case to the driftcli tests that seeds a Drift-Verify run status and a run counter written by driftengine.ResetRuns with operator true, calls the worktree's verbStatus, and asserts VerbStatus.OperatorSession is true and AtOperatorSession holds. Add the same seed with operator false and assert both are false. Put them as rows of one table-driven test, not two tests.
  - id: F3
    severity: LOW
    class: design
    location: internal/driftengine/verify.go:498
    origin: lens:code-correctness
    summary: The mode recorded in runs.json comes from a second d.Config() read in prepare, not from the table that is spawned, so a flip of verify_interactive between the two reads records a mode that disagrees with the session
    fix: In runAgent, have the tables closure keep the Interactive value of the table it returns, for example in a local variable set from tableFor's result, and make prepare pass that value to ResetRuns instead of reading d.Config(). Then prepare no longer needs Config and the recorded mode always equals the spawned table's Interactive.
  - id: F4
    severity: LOW
    class: consistency
    location: internal/driftengine/seats.go:3
    origin: lens:code-affected-set
    summary: The file header still says each drift agent is one non-interactive lead seat, which is false for the verify seat under verify_interactive
    fix: Reword line 3 of seats.go, which opens Each agent is one non-interactive lead seat whose one output is its report, so it says each agent is one lead seat whose one output is its report, interactive for the verify seat when drift.yaml sets verify_interactive. doc.go already says this.
  - id: F5
    severity: LOW
    class: scope
    location: internal/driftengine/verify_test.go:628
    origin: lens:code-tests
    summary: The claim that a resumed live agent keeps the mode it was spawned with, made in the change description and doc.go, has no test
    fix: Extend the existing resume case a live chair is resumed before any check, as a second row, by seeding ResetRuns with operator true, setting Config to VerifyInteractive false, running the entry, and asserting OperatorSessionOpen is still true and the run counter was not rewritten. Do not add a separate test function.
  - id: F6
    severity: LOW
    class: consistency
    location: internal/driftengine/seats.go:54
    origin: handler
    summary: The change description says the autonomous text renders as before, but the opening line lost the word autonomous and the acceptance paragraph now reads you judge intended where it did not
    fix: Make autonomousVerifyJudging open with the old sentence, Accept a drifted test with `lym drift accept <test> --reason <text>`, giving the reason the drift is intended., and keep its second line. In contracts/stencils/drift/drift-template-verify.md make the opening line read one autonomous session again only if the autonomous mode keeps it, otherwise correct the claim in _lym/landing/summary.md to say the opening line and the acceptance sentence are reworded.
  - id: F7
    severity: LOW
    class: decision
    location: internal/driftengine/verbs.go:83
    origin: handler
    summary: An operator session is allowed to re-accept a test the run's parent withdrew and to rule its own acceptances under scope full, which the board entry does not ask for and no Added after Discussion entry records
    fix: Leave the code in place and defer this finding to the run's parent, whose only sound change is recording the licence for operator-ruled acceptances and for re-accepting a withdrawn test with lym loom decision add, or refusing it. The fixer does not revert.
---

### [MEDIUM] PATTERN-parent-directive left stale by a second interactive role

**Location:** pattern/PATTERN-parent-directive.md:5
**Issue:** The entry says the discussion role's interactive questions are the one place a question reaches a person, and that they come from the `{{.mode_rules}}` marker. `VerifyTable` now fills the same marker with `operatorVerifyMode` under `verify_interactive`, sets `Table.Interactive`, and the seat engine then drops the parent directive's operator-ban line (`internal/seatengine/prompt.go:55`, `internal/parentdirective/parentdirective.go:36`). The diff changes no `PATTERN.md` or `pattern/` file, so a rule document now misdescribes the code.

### [MEDIUM] The operator mark's wiring into `lym drift accept` is untested

**Location:** internal/driftcli/wiring.go:69
**Issue:** `verbStatus` reads `OperatorSessionOpen` into `VerbStatus.OperatorSession`, and `Accept` records `by: operator` only through that field. No `driftcli` test touches `verbStatus` or the field; the only changed `driftcli` test line is the `ResetRuns` signature in `capture_integration_test.go:221`. The `verify_test` helper `sessionAccept` rebuilds the status by hand instead of calling the wiring, so a dropped field leaves every test green while a real operator acceptance records as agent and unruled.

### [LOW] Recorded mode can disagree with the spawned table

**Location:** internal/driftengine/verify.go:498
**Issue:** The table is built by `d.VerifyTable`, which loads the config, and `prepare` then loads the config again for `ResetRuns`. A change to `verify_interactive` between the two reads leaves `runs.json` with `operator` true for a non-interactive session or the reverse. A true flag on an autonomous session would make `Accept` record ruled acceptances that skip the parent's halt. The window is small, and one source for both removes it.

### [LOW] `seats.go` header contradicts the verify seat

**Location:** internal/driftengine/seats.go:3
**Issue:** The header reads that each agent is one non-interactive lead seat. `VerifyTable` sets `Interactive: cfg.VerifyInteractive` (`seats.go:238`), and the diff updated the same claim in `doc.go` but not here.

### [LOW] Resumed live agent keeps its mode: no test

**Location:** internal/driftengine/verify_test.go:628
**Issue:** The change description and `doc.go` state that a resumed live agent keeps the mode it was spawned with. It holds because `MultiLLMProducer.Call` skips `prepare` on a resume (`internal/shedadapters/multillm.go:106-111`). The resume case at `verify_test.go:628` seeds no operator `ResetRuns` and asserts nothing about `Operator`, so a change that rewrote the counter on resume would pass.

### [LOW] Autonomous text no longer renders exactly as before

**Location:** internal/driftengine/seats.go:54
**Issue:** The change description's Stencil bullet says the autonomous text renders as before. The stencil's opening line changed from one autonomous session to one session, and `autonomousVerifyJudging` begins `Accept a drifted test you judge intended with` where the old stencil began `Accept a drifted test with`. The rules the text carries are unchanged, so this is a mismatch between the claim and the diff, not a dropped rule.

### [LOW] Operator-ruled and withdrawn-test acceptance goes beyond the board entry

**Location:** internal/driftengine/verbs.go:83
**Issue:** The board entry asks for an interactive last drift step in which the operator decides and the agent records each acceptance or fix before the run goes on. The diff also makes an operator session's acceptance ruled under scope full, so the parent's decision halt is skipped, and lets it re-accept a test the parent withdrew. The decision record holds no `Added after Discussion` entry that licenses either, and the change description lists them as the writer's own design decisions. The code is coherent, and the call is the parent's to record or refuse.

## Rejected

- Lens code-tests, LOW: nothing asserts that the parent directive's operator-ban line is absent from the interactive verify prompt. The drop is `parentdirective.Directive`'s existing behaviour, pinned by `TestDirective_InteractiveOmitsOnlyBan` in `internal/parentdirective/parentdirective_test.go`, and the diff's own wiring is pinned by the `table.Interactive` assertion in `seats_test.go:255` together with `internal/seatengine/prompt_test.go:145`, which pins the chair spec for an interactive table.
- Lens code-tests point 19, an uncovered combination of an interrupted operator session and the mark: `CloseAgentSession` is pinned at `records_test.go:110-114` and the interrupted close at `verify_test.go:648`, and `OperatorSessionOpen` is `SessionOpen && Operator`, so the combination adds no behavior.
