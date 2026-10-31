---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/loomcli/decision.go:842
    summary: decision add on a beam has no test for the beam-orch caller rule, the member-pair notice fan-out or the project-record way-forward paths
    fix: In internal/loomcli/decision_test.go add rows that run the verb against a beam fixture (recipe with a Beam-Await row). Assert that decisionParent accepts the beam's nearest live orch, that decisionReceivers returns the live strands of the beam and of every memberPairs pair (and skips an absent pair worktree), and that decisionWayRecordRestore and decisionWayDiffers name `project/<slug>/decision-record.md` for a beam, not the discussion path.
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/loomcli/projectrecord.go:728
    summary: withProjectDirective is untested, so nothing pins that a member under a beam gets project_directive filled and a pair under no beam keeps its prompt byte-identical
    fix: In internal/loomcli/projectrecord_test.go add a table test over withProjectDirective. A pair under a beam with a record gets a non-empty project_directive naming the record path. A pair under no beam gets an empty value, and filling the DiscussionTable and DarnTable prompts with it equals the prompt filled with no project_directive key.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/loomcli/projectrecord.go:630
    summary: commitRunRecord (project branch), recordHistory and the project-vs-discussion choice at the wiring sites (wireLightweight, recordWriteDeps, arbiterDepsFor, landingDeps, beamDescribeDeps) have no test that fails if a site still reads _lym/discussion for a beam
    fix: In internal/loomcli/wiring_beam_test.go extend the pawl row to assert DecisionRecordPath, SupportLogPath, the Record-Write RecordPath and DiscussionDir, the arbiter prompt's record and the landing deps' DecisionRecordPath equal the loomengine.ProjectDecisionRecord paths and that PatternRecordPath is empty. Add one commitRunRecord test that a project record commit stages loomengine.ProjectDirRel(slug).
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/boardengine/runlock.go:65
    summary: a run sent back to Preflight with goto makes the project body editable again while Record-Write keeps the existing record, so an edit made then never reaches the record
    fix: Either judge ProjectRecordHeld from the existence of the project record as well as the status, or make the body refusal in checkRunLocks name the same decision-add route for a beam that has run past Record-Write once. If neither is wanted, state in the ProjectRecordHeld doc comment that a goto to Preflight lifts the lock and the existing record is kept.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/loomcli/decision.go:842
    summary: preRun now reads the run's seed for every decision add and aborts with no way forward when the seed cannot be read, where it used to surface only as a refusal inside the hasPlan closure
    fix: In preRun, when seedRecipe fails, emit the error with a way forward (for example restore the run's seed or re-run `lym loom start`) in the message passed to output.Err, matching the other decision refusals' way-forward wording.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/loomengine/projectdirective_test.go
    summary: the RecordsRenderAsBulletsInOrder subtest asserts the directive stencil's static prose (the heading and the decision add line), which the rubric's test economy rule names as a finding
    fix: In internal/loomengine/projectdirective_test.go delete the loop over the wanted text strings in RecordsRenderAsBulletsInOrder. Keep the bullet-order, banner-stripped and token-substituted assertions.
  - id: F7
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: internal/loomcli/decision.go:925
    summary: a beam with no resolvable orch is refused with wording that says the caller is a task session
    fix: In decisionParent, when record.project and NearestLiveOrch returns an empty Parent, return a refusal that names the beam's orch as the one allowed caller and the way forward to start it with `lym beam start <beam-slug>`.
  - id: F8
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/loomengine/discussionpath_test.go
    summary: the ProjectDecisionRecord and ProjectDir assertions against the Rel forms repeat the assertions just above them with no new behaviour
    fix: In internal/loomengine/discussionpath_test.go remove the repeated assertions against ProjectDecisionRecordRel and ProjectDirRel and keep the two-beams-differ check.
---

### [MEDIUM] decision add on a beam is untested

**Location:** internal/loomcli/decision.go:842
**Issue:** The decision record makes the beam-orch caller rule and the notice to every live member run the core of `decision add <beam-slug>`. `decisionParent`, `decisionReceivers` and `memberPairs` have no test, and the only test change is the new `recordUnderLym` field on the fake. A regression in any of them passes the suite.

### [MEDIUM] withProjectDirective is untested

**Location:** internal/loomcli/projectrecord.go:728
**Issue:** The decision record asks that each member's Discussion and writer read the project record, and the change description claims a run under no beam gets a byte-identical prompt. Only `ProjectRecordDirective` is tested; the wiring that fills the marker for `DiscussionTable` and `DarnTable` and the optional-marker lists are not.

### [LOW] Record choice and project commit untested at the wiring sites

**Location:** internal/loomcli/projectrecord.go:630
**Issue:** `TestRunRecordOf` pins the choice function, but no test pins that each consumer site uses it. A site that read the discussion path for a beam would pass, and `commitRunRecord`'s project pathspec and message are unchecked.

### [LOW] A goto to Preflight lifts the body lock over an existing record

**Location:** internal/boardengine/runlock.go:65
**Issue:** The lock is judged from the status alone. At Preflight the body is editable, and Record-Write then writes only a missing file, so a body edit made in that window never reaches the record that binds the members.

### [LOW] decision add aborts on an unreadable seed with no way forward

**Location:** internal/loomcli/decision.go:842
**Issue:** `preRun` reads the seed for every call now. An unreadable seed used to become an ordinary refusal inside the `hasPlan` closure and now aborts the verb with a message that names no next step.

### [LOW] Test asserts static stencil prose

**Location:** internal/loomengine/projectdirective_test.go
**Issue:** The `RecordsRenderAsBulletsInOrder` subtest asserts text of the directive stencil, which is static prose. The rubric names a new assertion on a stencil's static prose as a finding.

### [NIT] Misleading refusal for a beam with no live orch

**Location:** internal/loomcli/decision.go:925
**Issue:** The caller refusal for a beam whose orch is not live reads as if the caller were a task session.

### [NIT] Repeated path assertions

**Location:** internal/loomengine/discussionpath_test.go
**Issue:** Two new assertions restate the neighbouring ones and add no coverage.

## Rejected

- Record written by Record-Write instead of by `lym beam create` itself (lens: handler): the summary states why the verb makes no pair, and the entry's observable outcome holds, a record with the body verbatim under `## Goal` before Beam-Up and no body edit after it.
- The `Co-Authored-By` trailer on `3c8d5a3c` (lens:code-affected-set): the commit that moved landing commits to a credit block governs landing commits, not the branch's working commits.
- `landingdeps.go` and `arbiterprompt.go` still key on `RecipePawl` (lens:code-affected-set): the lens judged the behaviour equivalent today, so there is no defect to fix.
- A beam halted at Record-Write has its body locked while `decision add` refuses (lens:code-correctness): the refusal names the re-step as the way forward and the lock at Record-Write is the design the summary states.
