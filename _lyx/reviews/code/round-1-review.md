---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/stencils.go:536
    summary: "roleOpeningStencils has no entry for the new plan-fixer role, so the parent and edit directive enforcement test never checks webster-template-plan-fix"
    fix: "In contracts/stencils/stencils.go add the entry plan-fixer mapped to the one stencil webster-template-plan-fix to the roleOpeningStencils map, beside verify-fixer, so parentdirective_test.go checks both markers in it."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/websterengine/runwrites_test.go:63
    summary: "D7 behaviour that recordedSessions and loadRunWrites leave plan fixer sessions out of the run audit has no test; the existing row seeds only VerifyFixers"
    fix: "In internal/websterengine/runwrites_test.go extend the subtest over recordedSessions and the audited sessions (the one seeding VerifyFixers) so its State also carries PlanFixers with a distinct SessionID, and assert that session is never listed or audited."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/webstercli/batchpause_test.go:143
    summary: "New fix-plan refusals have no test reaching them, the covering-pause refusal and the transient way forward refusals of FixPlanSpawnOrAttach, against PATTERN-refusal-way-forward"
    fix: "Add a fix-plan case beside the fix-verify case at internal/webstercli/batchpause_test.go:143, and add rows to the refusal table in internal/websterengine/planfixer_test.go for HeadSHA failing, DirtyPaths failing, Starter.Start failing and FindRun failing, each asserting the transient way forward text and that no record is appended."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: contracts/stencils/markers_test.go:50
    summary: "The stencil marker table and the per-stencil friction directive table gain no row for webster-template-plan-fix"
    fix: "Add a webster-template-plan-fix row, with its required markers record_path, reply_path, worktree_root, plan_dir, specs_dir and edit_directive, to the table in contracts/stencils/markers_test.go near line 50, and a plan-fix friction on and off case to the table in internal/websterengine/template_test.go near line 962."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/websterengine/planfixer_test.go
    summary: "The spawn subtest asserts static stencil prose, the rebaseline, restore-plan and verdict strings, which Test economy forbids"
    fix: "In the want list of the spawn subtest in internal/websterengine/planfixer_test.go keep the dynamic values (record path, reply path, specs path, ly:orch) and delete the four static strings lyx webster rebaseline --card NN, lyx webster restore-plan, verdict: fixed and verdict: plan_writer."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-pattern-fit
    location: internal/websterengine/conflictrecord.go:67
    summary: "ReadConflictRecord re-parses the finding lines inline beside readFindingLines, and its Cause field is parsed but never read"
    fix: "In ReadConflictRecord take the findings from readFindingLines instead of re-parsing the dash prefix, and drop the Cause field and its parse since nothing reads it."
  - id: F7
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/webstercli/wiring.go:344
    summary: "Comments listing the agent-spawning, pause-covered verbs omit fix-plan"
    fix: "Add fix-plan to the verb lists in the comments at internal/webstercli/wiring.go:344 and internal/webstercli/batchpause.go:1."
---

### [MEDIUM] roleOpeningStencils lacks the plan-fixer role

**Location:** contracts/stencils/stencils.go:536
**Issue:** Every spawned role has an entry in `roleOpeningStencils`, which `parentdirective_test.go` reads to enforce the parent and edit directive markers (`PATTERN-parent-directive`, enforced by a test over the stencils). The new `plan-fixer` role has none, so a later edit dropping a marker from `webster-template-plan-fix.md` would pass. I confirmed the map in the file: `verify-fixer` and `webster-recovery` are present, `plan-fixer` is not.

### [MEDIUM] D7's audit exclusion is untested

**Location:** internal/websterengine/runwrites_test.go:63
**Issue:** D7 requires the plan fixer's session to stay out of `recordedSessions`. The only test of that list seeds `VerifyFixers`. A regression that listed plan fixers would turn the fixer's one legitimate write, to the plan directory, into a correctness violation at run exit, and no test would fail. The behaviour is added by the diff by omission and carries no coverage.

### [LOW] New refusals lack reaching tests

**Location:** internal/webstercli/batchpause_test.go:143
**Issue:** `fix-plan` refuses first under a covering pause through `refuseCovered("fix-plan", ...)`, and `FixPlanSpawnOrAttach` returns transient way-forward refusals for `HeadSHA`, `DirtyPaths`, `Starter.Start` and `FindRun` failures. `PATTERN-refusal-way-forward` asks for a test that reaches each new refusal; the `fix-verify` pause case exists, the `fix-plan` one does not, and the engine refusal table has only the five static refusals.

### [LOW] Stencil marker and friction tables not extended

**Location:** contracts/stencils/markers_test.go:50
**Issue:** The new stencil's required and optional markers are not pinned in the top-level marker table, and the friction directive on and off table in `internal/websterengine/template_test.go` has no plan-fix case although `RenderPlanFixPrompt` runs the same `friction.Directive` path.

### [LOW] Test asserts static stencil prose

**Location:** internal/websterengine/planfixer_test.go
**Issue:** The `want` list of the spawn subtest asserts `lyx webster rebaseline --card NN`, `lyx webster restore-plan`, `verdict: fixed` and `verdict: plan_writer`, fixed text of the stencil. Test economy names a new assertion on a stencil's static prose as a finding.

### [LOW] Duplicate record parser and unused Cause

**Location:** internal/websterengine/conflictrecord.go:67
**Issue:** `ReadConflictRecord` re-implements the dash-prefix finding parse that `readFindingLines` in the same file already does, so the `fix-plan` bound and `ClaimConflictRoute` can read the same record two ways. `ConflictRecord.Cause` is parsed and read nowhere.

### [NIT] Stale verb lists in comments

**Location:** internal/webstercli/wiring.go:344
**Issue:** The comments at `wiring.go:344` and `batchpause.go:1` name `run`, `recover-batch` and `fix-verify` as the verbs that spawn an agent or refuse under a pause; `fix-plan` now does both (`fixplan.go:87`).

## Rejected

- Lens correctness F1, the dirty-at-spawn narrowing in `codeTouchedSince`: deliberate and documented in `doc.go`, `planfixer.go` and the summary; a pre-dirty worktree would otherwise overrule every fix.
- Lens correctness F2, a git read error after the fixer is done leaves the record non-terminal: recoverable by re-running `fix-plan`, whose error names the way forward.
- Lens correctness F3, no restore when the fixer dies or is overruled: D3 makes restore the fixer's own step, and the unchanged plan-review route handles the leftover plan.
- Lens correctness F4, `PlanFixers` persisting across a plan-review reroute: D6 states the bounds as per run; the effect is conservative and ends in the existing reroute.
- Lens pattern-fit F1, `restore-plan` is operator-audience yet the stencil sends the fixer to it: `AudienceAnnotation` only filters the help index (`internal/clihelp/index.go:54`), and no execution gate on it was found.
- Lens pattern-fit F4, `filepath.Abs` and stale-reply `os.Remove` errors without a way forward: OS-level failures no caller input reaches, with no differing action to name.
- Lens tests F3, no test that `PlanFixers` loads absent as nil: the field is a plain `omitempty` slice decoded by `encoding/json`, with no behaviour of its own to pin.
- Lens tests F7, the `t.Setenv` comment and the re-covered no-record row: the CLI row adds the exit code and envelope the engine row lacks, and the comment is a style point of the sibling tests.
- Handler checks that held: the plan edit reaches the weft by `fabricSync`, so a fixed verdict does not move the warp HEAD that D7 compares; the `awaitColdFixer` extraction matches the removed `FixVerifyAwait` body step for step; the Master stencil rung runs `fix-plan` backgrounded, re-begins on `fixed`, ends stuck otherwise and keeps `plan_drifted` stuck; `webster-spec.md` has no verb or conflict text to update; the loom recipe, shed adapter and plan-review instructions are untouched.
