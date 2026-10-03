---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md D2 (plankit) and Scope (last in-scope bullet)
    summary: plankit's glyph-plan writer takes over #327 section 2's plan-fixture work, which the board assigns to test-suite-derived-guards
---

### [BLOCKING] plankit duplicates part 2's plan-fixture writer

**Location:** `_lyx/discussion/decision-record.md`, D2 (`internal/testkit/plankit`), the last in-scope bullet of `## Scope`, Q7 in the support log, and the acceptance criterion "`plankit` replaces the glyph-plan and file-tree fixture copies in `loomshed` and `loomcli`".

**Issue:** #327 section 2 (a plan-directory fixture writer under `internal/testkit/`) names loomcli's `glyphPlanFixture` and `glyphRepoPlanFixture` and planglyph's `writePlanFixture` among the helpers it replaces.
The board puts #327 in `test-suite-derived-guards`, and this task's own Scope lists #327 as out of scope.
D2's `plankit` one-card glyph-plan writer covers the same loomcli/loomshed glyph-plan copies, so part 1 would build a second plan-fixture kit that part 2 then has to merge into or replace.
That contradicts the other board entry and widens a task the operator already considers very large.
Removing the old-policy citations in `loomcli/validate_test.go`, `loomcli/parity_test.go` and `loomshed/fixture_test.go` does not need the fixtures shared: #324 asks only that each citation is removed when its code is converted.

**Fix:** Drop `plankit` and Q7 from this task.
Leave the glyph-plan and file-tree fixtures in `loomcli`, `loomshed` and `planglyph` for #327 section 2.
Narrow the acceptance criterion "no comment cites the duplicate-test-helpers decision" so it excludes those plan-fixture sites, or reword their citations without converting the code.
Record the sites as a note for `test-suite-derived-guards`.
Everything else is in scope, and D13's invariant changes stay within the operator directive.
