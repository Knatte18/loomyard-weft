# Card 10 — darn-review-scope

**Edit:**
- `contracts/stencils/loom/loom-rubric-darn-review.md`
- `contracts/recipes/darn-recipe.yaml`
- `contracts/stencils#wordingClaims`
- `internal/loomrecipe#TestNewDarn_RowsMatchTheDarnTable`

**Intent:** Make the Darn-Review perch measure the diff against the board entry as amended by the record's `Added after Discussion` entries, treat the writer's own record sections as a claim, and settle an unrecorded parent correction by deferring it to the parent instead of reverting it (D7).

In `loom-rubric-darn-review.md`, the measuring stick becomes the board entry as amended by the `Added after Discussion` entries of `_lyx/discussion/decision-record.md`: an entry by the parent, the operator or the arbiter wins over the board body where they differ.
The record's other sections are the writer's own claim, made before coding and reviewed by no one, checked against the board entry the way the rubric checks the change description, never the measuring stick.
"Do not flag" adds the record's added entries to the measuring stick that is never a finding's subject.
"Scope against the board entry" measures against the amended entry.
A new "Also flag" bullet, "An unrecorded parent correction": a deviation from the amended board entry that the change description names as an unrecorded parent correction is never fixed by revert; it is filed as class `decision` at severity MEDIUM, and its fix text tells the fixer to leave the code in place and defer the finding, since its only sound change, recording or refusing the correction, is the run's parent's with `lyx loom decision add` and lies outside the fixer's write surface; the fix text names the parent.
`burler-step-fix.md`'s existing rule, that "a finding whose only sound change leaves the surface is deferred with that reason, never disputed", is what the Darn-Burler-Fix fixer follows on that fix text, so the fix stencil's text is unchanged and a claim pins it.

In `darn-recipe.yaml`, the Darn-Burler-Review row's `fasit` instructions say the same: the answer key is the board entry read with `lyx board get`, as amended by the `Added after Discussion` entries of `_lyx/discussion/decision-record.md`, which win where they differ; the record's other sections are the writer's claim; and a deviation the change description names as an unrecorded parent correction is a `decision`-class MEDIUM finding whose fix text tells the fixer to leave the code and defer to the parent.
The Darn-Bouncer row's `artifact_paths` comment, which calls the board entry the measuring stick, says the amended board entry.

The fasit text uses those words: `claim`, `decision`, `MEDIUM` and `leave the code`.
`TestNewDarn_RowsMatchTheDarnTable` additionally reads the parsed recipe's Darn-Burler-Review `profile.fasit.instructions` and asserts it names `Added after Discussion`, `claim`, `unrecorded parent correction`, `decision`, `MEDIUM` and `leave the code`, and its doc says so.
The `burler-step-fix.md` row of `wordingClaims` gains a must claim on `is deferred with that reason, never disputed`, why: the Darn-Burler-Fix fixer defers an unrecorded parent correction to the parent instead of reverting it.
A new `wordingClaims` row for `loom-rubric-darn-review.md` over `LoomRubricDarnReview` claims `Added after Discussion`, `_lyx/discussion/decision-record.md`, `a claim`, `unrecorded parent correction`, `` `decision` ``, `MEDIUM`, `leave the code in place` and `lyx loom decision add`.

**ImpactSummary:** the Darn-Review rubric and its review row's answer key change wording; no Go behavior changes, and the one recipe test gains an assertion.

**Commit:** `10: Darn-Review measures against the amended board entry and defers an unrecorded correction`
