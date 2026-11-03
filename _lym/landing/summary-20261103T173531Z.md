# Drift's regression check reaches the pull request and the landing commit

Drift-Verify ran the regression compare, but its result never reached the change description or the pull request.
A description written before Drift-Verify could still ask the reviewer to "run the regression harness compare by hand", and the PR-Review agent repeated that request to the operator, though the compare had already passed.

## What changes

- **Drift-Verify records each compare.**
  The scope record (`_lym/drift/scope.json`) gains `baseline_head`, `compared_head`, `compared_at` and `compare_passed`.
  The row writes them after its own compare, whether it passed or showed drift, and again when the Drift-Verify agent ends Done, which happens only after its final compare passed.
- **`driftengine.Verdict` renders the check.**
  It renders the scope record as a `## Regression check` section: the scope (`full`, `partial` with its skipped tests, or `off`), the commit the baseline was captured at, the last compare's result or the waiver's reason, and each accepted drifted test with who accepted it and why.
  It renders nothing when no scope record exists, and it checks for the file before reading it, so a recipe with no Drift rows gets no `.lym/drift/` directory.
- **The landing appends the section.**
  `landingshed.Deps` gains a `DriftVerdict` seam, which `loomcli`'s `landingDeps` fills from the task's scope record at call time.
  PR-Review puts the section after the description's body and before the credit block when it opens or refreshes the pull request.
  Finalize does the same in the landing commit message and the GitHub merge message.
  A failed read is an error from the row, the same as a failed `EnginesRun` read.
- **The prompts show the check.**
  The Describe stencil (`landing-template-describe`) and the PR-Review stencil (`landing-template-pr-review`) take a new `drift_verdict` marker holding the section, or "No regression check ran for this task." when none did.
  Describe is told that lym appends the section itself, and to drop or rewrite any item to check by hand that the check settles.
  The PR-Review agent is told the check already ran and not to ask the operator to repeat it.
- Docs: the package docs of `driftengine` and `landingshed`, and `loom-final-summary-spec.md`'s consumers section.

## Design decisions

- The record's Expected says Describe adds the outcome to the summary.
  Here Go appends the section when the description is published and landed, and `summary.md` never holds it.
  With the Code-Review perch off, Describe's entry gate skips the row and the writer's description, written before Drift-Verify, is what lands.
  Appending in Go puts the check on the pull request in that case too, and no agent can drop or reword it.
  Describe still sees the check, so it can drop a stale request for a manual compare.
- With the perch off, a request for a manual compare in the writer's own description stays in place, and the appended section below it answers it.
  The darn writer's stencil is unchanged, since the writer runs before Drift-Verify and cannot know its result.
- A run already past Drift-Verify when this lands has no compare fields in its scope record.
  Its section names the scope and acceptances and reads "Compare: not run."

## To check by hand

- On a hub with Drift enabled, run a darn task with a regression scope through PR-Review.
  Confirm the pull request's body ends with the `## Regression check` section before the credit block, and that the PR-Review agent's opening prompt shows the section.
