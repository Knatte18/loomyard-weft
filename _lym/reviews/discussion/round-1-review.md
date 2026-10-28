---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1, Constraints (lens prose rule), Acceptance criteria D1
    summary: board-scope leaves every review-* and fix-* fan, so the template rule and the lens test class it as an emphasis lens that must end with the emphasis line, contradicting D1's ends-in-its-list prose and the procedure-lens Constraint
    fix: In D1, decide which rule gives and state it. Either have board-scope's prose keep its own list and then end with the line "Report anything else you notice too — emphasis, never exclusion.", as leanness does, and reword the Constraint to "procedure lenses of the review-* and fix-* fans end in their own list; every other lens, board-scope included, ends with the emphasis line"; or state that the template header and the emphasis-line exemption in internal/lenses/config_test.go widen to name board-scope, and add that to D1's acceptance line. Keep the D1 acceptance line in step with the choice.
  - id: F2
    severity: LOW
    class: design
    origin: lens:goal-scope
    location: _lym/discussion/decision-record.md D6
    summary: D6 makes light's loom block solo but leaves speed's loom block on the template fan, so the lightest profile fans its code review while the next one up runs solo, and the pick is not stated
    fix: In D6, state the speed pick. Either add loom.code_reviewer_fan "" to speed's loom block, beside light's, and name speed in the D6 acceptance line and in Auto-mode assumptions; or add one sentence to D6 saying why speed's loom block keeps the fan, and list that pick under Auto-mode assumptions.
  - id: F3
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5 (retired keys sentence), Acceptance criteria D5
    summary: Loom's own MigrateConfig renames the retired webster_reviewer_fan (and webster_fan through it) onto code_reviewer_fan before checkLocalDocument runs, so D5 admits those routes too, but D5 names only the darn key moves
    fix: In D5's sentence on retired keys, add that loom's retired webster_reviewer_fan and webster_fan, which loom's MigrateConfig renames onto code_reviewer_fan, are admitted with it under the same bound; extend the D5 acceptance line to cover a run_settings or profile block holding webster_reviewer_fan.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D7, second bullet
    summary: D7 names loom's template.yaml key comment and internal/configcli/doc.go as text that calls the reviewer fan keys closed to the local layer, but neither holds such text
    fix: In D7's second bullet, drop loom's template.yaml key comment and internal/configcli/doc.go from the list, leaving docs/overview.md's loom config line and loom's config.go comments.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md Open risks, second bullet
    summary: The risk says a hub fan naming a deleted lens fails every loom config load, but LoadConfig resolves only the six reviewer and fixer fan keys; a fan named only by discussion_lenses or plan_lenses fails later, when the writer lead resolves it
    fix: In the second Open risks bullet, say that a fan one of the six reviewer or fixer fan keys names fails every loom config load, and that a fan named only by discussion_lenses or plan_lenses fails when the Discussion-Write or Plan-Write lead resolves it; the operator's way forward stays the same.
  - id: F6
    severity: NIT
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D6, second sentence
    summary: speed, solid and wide take whatever the hub's loom stack resolves for code_reviewer_fan, which is the template value only when no layer sets the key
    fix: In D6, change the clause saying speed, solid and wide take the template's review-code fan so it says they take the hub's code_reviewer_fan, which is the template's value where no layer sets the key.
---

### [MEDIUM] board-scope is no procedure lens by the template's own rule

**Location:** `_lym/discussion/decision-record.md` D1, Constraints, Acceptance criteria D1

**Issue:** The template header (`internal/lenses/template.yaml:33-36`) calls only the lenses of the `review-*` and `fix-*` fans procedure lenses.
`TestConfigTemplate` builds its exemption from the emphasis line from those fans alone (`internal/lenses/config_test.go:296-300`), and every other lens except `generic` must end with "Report anything else you notice too — emphasis, never exclusion." (`:341-343`).
`goal-scope` and `board-fit` are exempt today only because they sit in `review-discussion`.
D1 takes that fan away from them, so `board-scope` sits only in `twill-discussion` and the dormant `discussion` fan, and the test refuses a prose that "ends in its list".
`leanness`, the other twill-only lens, ends with the emphasis line (`template.yaml:132-137`).
The record never says which rule gives, so the plan writer meets a contradiction between the decision, its Constraint and an existing test.
Four lenses raised this on their own.

### [LOW] speed keeps a fanned loom code review below a solo light

**Location:** `_lym/discussion/decision-record.md` D6

**Issue:** `speed`'s loom block sets `code_max_bounces: 1` and no fan (`internal/runprofile/template.yaml:16-21`), so its one code round fans out over `review-code`, while `light`'s loom block runs solo under D6.
The entry names `light`, `docs`, `solid` and `wide`, and leaves `speed` open.
The record makes a pick by not stating one, and it is not listed under Auto-mode assumptions.

### [LOW] D5 leaves loom's own retired fan keys out of its route list

**Location:** `_lym/discussion/decision-record.md` D5, Acceptance criteria D5

**Issue:** `PrepareRunSettings` runs `module.Migrate` before `checkLocalDocument` (`internal/configreg/runsettings.go:72-84`), and loom's `MigrateConfig` renames `webster_reviewer_fan` to `code_reviewer_fan` (`internal/loomengine/config.go:1035`), with `webster_fan` reaching it through `webster_reviewer_fan`.
After D5 those keys are admitted from a profile block, `run_settings` or the local layer, the same as the darn keys D5 names.
The bound holds for them, but D5 states the route list as if it were complete, and the attack-surface rule asks for every route to be named.

### [LOW] D7 names two edit sites that hold no such text

**Location:** `_lym/discussion/decision-record.md` D7, second bullet

**Issue:** `internal/loomengine/template.yaml`'s fan key comments never mention the local layer, and `internal/configcli/doc.go` names the fan keys only for their value shape (line 56).
Only `docs/overview.md:583` ("all six closed to the local layer") and loom's `config.go` comments hold the text D7 changes.

### [LOW] Open risk 2 overstates where a deleted lens fails

**Location:** `_lym/discussion/decision-record.md` Open risks, second bullet

**Issue:** `LoadConfig` calls `lenses.ValidateFanKeys` with `fanKeys(cfg)` only (`internal/loomengine/config.go:1225`), the six reviewer and fixer keys.
`discussion_lenses` and `plan_lenses` (`:403-409`) are not resolved at load, so a hub that tuned `twill-discussion` to name `goal-scope` loads cleanly and fails at Discussion-Write instead.
The operator's step is right either way; the stated reach is not.

### [NIT] D6 says the template value where the hub's value applies

**Location:** `_lym/discussion/decision-record.md` D6

**Issue:** A profile that sets no fan key takes the hub's loom stack, which equals the template's `["review-code", ""]` only where no layer sets `code_reviewer_fan`.
It holds on this hub (`_board/_lym/config/loom.yaml` sets only `discussion_reviewer_fan`), but not in general.

## Rejected

- goal-scope F2, D1 widens the entry's "only what contradicts the board entry or another board entry" to runs and parent commits: rejected. The entry merges `board-fit`, whose run and parent-commit reach is where its gating finds come from, and the rubric's `decision` class names a conflict with another run and a landed parent commit, so carrying that reach is the merge the entry asks for, not new scope.
- goal-scope F3, `decision` question on whether the profile line is met when only the template changes: rejected. The record already names the operator step in Open risks, the run cannot write Bolt config, and no choice is left that the parent could make inside this run; the support log shows the writer kept it for the parent.

## Focus directive verdicts

- D5: every route now has a stated bound except loom's own retired keys (F3). The claim that a wrong name fails at load holds: `LoadConfig` runs at `internal/loomcli/wiring.go:378` for loom, darn and pawl runs, and the `ValidateFanKeys` refusal names the key and the fix.
- D6 and Open risks: the solo value reaches a seeded pair on a newly seeded hub. Seed-Child writes `[""]`, `ValidateFanKeys` skips it, `fanSetting` turns it into a nil list, and `ReconcileAll` reconciles the local file with a nil strip. No fact is left for Plan or an implementer to find out. On this hub the live `profiles.yaml` is the operator's, as the record says.
- D1 and D2 lens prose: `repo-invariants` keeps the template's rules. `board-scope` does not, because it is not a procedure lens under the template's rule (F1). Neither lens tells a fork to ignore anything. `board-scope`'s `decision` triggers (doubt about the task, an entry plainly outdated by landed work, a conflict the writer cannot settle) agree with the rubric's definition: the rubric's "contradicted by another entry or run" and "design conflict with another run" fall under a conflict the writer cannot settle, and the rubric's own "a choice the writer can make is never `decision`" is the same cut. Not a defect.
- D3 and D4: nothing dropped is a requirement the plan writer needs. The two names D3 drops appear in no rubric (`loom-rubric-code-review.md` carries neither). D4 leaves "Test economy" with the review lead, and the rubric is unchanged.
- No Decision edits this task's own board entry, and none files or promotes a task.
