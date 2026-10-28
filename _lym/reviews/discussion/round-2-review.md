---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D2, the Constraints lens-prose bullet and the second D1 acceptance line
    summary: D2 takes way-forward out of review-plan, its one review-* fan, so the record's own rule and TestConfigTemplate's computed exemption require it to end with the emphasis line, which its unchanged prose does not
    fix: In D2, after "`way-forward` stays defined and stays in `twill-plan`", add the sentence "Its prose gains the emphasis line after its list, as the other `twill-plan` lenses end, since it leaves the only procedure fan that exempted it." In the D2 acceptance line, after "`way-forward` stays defined", add "and its prose ends with the emphasis line". Leave the Constraint and the D1 acceptance line as written; they then hold for `way-forward` too.
  - id: F2
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5, third Bound bullet
    summary: D5 frames an agent's reach through its pair's local layer as thinning the code review to solo, but the key also lets it re-point Code-Review at any fan lenses.yaml defines, such as plan or twill-discussion, and the record does not state that substitution's floor
    fix: In D5's third Bound bullet, replace "An agent can thin its own run's code review to solo through its pair's local layer;" with "An agent can thin its own run's code review to solo, or point it at any other fan `lenses.yaml` defines, through its pair's local layer; either way the round keeps the reviewer's whole-diff read under the code-review rubric, which is the floor;".
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:goal-scope
    location: _lym/discussion/decision-record.md Auto-mode assumptions
    summary: D1's review-discussion pick, dropping goal-scope and board-fit with no replacement, is a choice the entry does not name, yet Auto-mode assumptions list only the dormant discussion fan pick
    fix: >-
      Under Auto-mode assumptions, after the D1 line, add the bullet "- D1: `review-discussion` drops `goal-scope` and `board-fit` with no replacement, which the entry does not name."
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5, the retired-key sentence
    summary: loom's MigrateConfig renames webster_reviewer_fan and webster_fan before the check only on the run_settings and profile route and in a reconcile of the local file; a hand-edited local file is checked raw at load and refused as not in the template until a reconcile migrates it
    fix: In D5, replace "which loom's `MigrateConfig` renames onto `code_reviewer_fan` before the check" with "which loom's `MigrateConfig` renames onto `code_reviewer_fan` before the check in `PrepareRunSettings` and in a reconcile of the local file; a hand-edited local file holding one is refused at load until a reconcile migrates it, as for every retired key".
---

### [MEDIUM] `way-forward` loses its emphasis-line exemption under D2

**Location:** `_lym/discussion/decision-record.md` D2, the Constraints lens-prose bullet and the second D1 acceptance line.

**Issue:** `TestConfigTemplate` builds its emphasis-line exemption from the `review-*` and `fix-*` fans of `wantOrders` (`internal/lenses/config_test.go:296-300`) and fails every other lens but `generic` that does not end with the line (`:335-337`).
Today `way-forward` is exempt only through `review-plan` (`internal/lenses/template.yaml:411-414`), and its prose ends with "Your list: per refusal, the next step, progresses or not." and no emphasis line (`:288-293`).
D2 removes it from `review-plan` and keeps it in `twill-plan` alone, unchanged.
That puts it under the Constraint's "every other lens ends with the emphasis line", and the second D1 acceptance line requires the test to still fail on it, so the template D2 describes fails the test the record keeps.
This answers the round's first focus question.
D1, its Constraint and its acceptance line agree with each other and with the header at `template.yaml:33-36` on `board-scope`.
The one rule they state then breaks on a lens D2 moves.
The `twill-plan` siblings `coverage`, `row-validity` and `forced-edits` end with their list followed by the emphasis line, so appending the line to `way-forward` keeps the one rule intact and needs no wider exemption.
Three lenses found the same defect independently.

### [LOW] D5's local-layer bound names solo but not a fan swap

**Location:** `_lym/discussion/decision-record.md` D5, third Bound bullet.

**Issue:** `ValidateFanKeys` checks only that a name resolves in `lenses.yaml` (`internal/lenses/fankeys.go:65-78`).
So once the key is `CategoryNone`, a pair's local layer can point `code_reviewer_fan` at `plan`, `twill-discussion` or `standard` as well as at `""`.
The first bullet states that any defined fan is admissible.
The third bullet states an agent's reach only as "thin … to solo", and the attack-surface question asks the decision to say what the new route lets through.
The weakest outcome is still a solo round with the reviewer's whole-diff read under the rubric, so this is a gap in wording, not a hole in the guard.
The bound-breaker fork rated it MEDIUM; I lowered it to LOW because the floor already holds and the first bullet already bounds the value.

### [LOW] Auto-mode assumptions omit D1's `review-discussion` pick

**Location:** `_lym/discussion/decision-record.md` Auto-mode assumptions.

**Issue:** The board entry names only `twill-discussion` for the merge.
D1 also leaves `review-discussion` with no replacement for the deleted pair, a choice the support log records as auto-pick Q3.
Auto-mode assumptions list the dormant `discussion` fan pick but not this one.

### [NIT] Retired loom fan keys are renamed only where Migrate runs

**Location:** `_lym/discussion/decision-record.md` D5, the retired-key sentence.

**Issue:** `PrepareRunSettings` runs `module.Migrate` before `checkLocalDocument` (`internal/configreg/runsettings.go:73-84`), and the config reconcile runs Migrate over a sparse local file.
`Load` checks a raw local file with `ValidateLocal` (`internal/configengine/layers.go:476-481`), which refuses `webster_reviewer_fan` as "not in the loom template" (`:275-277`).
The D5 acceptance line's "moved onto it" already reads as after migration.
D5's prose sentence, "before the check", names no route, so a plan writer could pin a raw local-file row that fails.

## Focus departures

None.
The focus questions were settled as follows.
The first, on lens prose: F1.
The second, on fan membership: no defect.
`speed`'s loom block, `light`'s three blocks and `docs`' darn block run solo.
`speed`'s darn and pawl blocks run no code round.
`solid` and `wide` take the hub value.
`docs` holds no loom or pawl block, and `runprofile.Settings` refuses a loom or pawl task that names it (`internal/runprofile/runprofile.go:83-91`).
Auto-mode assumptions list D6's two picks, `light`'s loom block and `speed`'s loom block, and every other D6 line follows from the entry or from `code_max_bounces: 0`.
The third, on routes: D5 names the profile block, `run_settings`, the pair's local layer, darn's key moves (`internal/configreg/keymove.go:30,32`) and loom's `retiredPerchKeyRenames` (`internal/loomengine/config.go:1035`, reached from `webster_fan` through `:1022`).
The `merge_fan` chain ends at a retired key and never reaches `code_reviewer_fan`, so the list is complete.
The bounds are F2 and F4.

## Rejected

- None of the forks' findings were rejected; the duplicate `way-forward` reports from the walkthrough and bound-breaker lenses are merged into F1.
