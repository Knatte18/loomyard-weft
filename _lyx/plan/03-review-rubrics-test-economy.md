# Card 3 — review-rubrics-test-economy

**Prosa:**
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils/loom/loom-rubric-webster-review.md`

**Edit:**
- `contracts/stencils/claims_test.go`

**Intent:** Make both review loops judge granularity by public surface and flag redundant tests (D3, D5), and move the rubric claims onto the new wording (D9).

`loom-rubric-plan-review.md`, `## Also flag`:

- The **Granularity** bullet: "One card per independently reviewable/testable unit" becomes "One card per independently reviewable unit"; "an independently testable symbol gets its own card even when one card is its only consumer" becomes: a symbol that is part of a module's public surface, or reused across packages, gets its own card even when one card is its only consumer, and a helper serving one surface is bundled into that surface's card.
  The private-supporting-type clause stays.
- A new **Test economy** bullet: a card that plans a new test function or file for behavior an existing test already covers, or one test per symbol, is a finding.
  The finding names the existing test that covers the behavior, or, for a per-symbol test, the public-surface test that covers the symbol; a vague "seems redundant" is not a finding.
  The bullet points at `PATTERN-test-economy` for the rule and restates nothing beyond this finding condition.

`loom-rubric-webster-review.md`, `## Also flag`: a new **Test economy** bullet pointing at `PATTERN-test-economy`, listing each finding case with what grounds it:

- a new test that duplicates an existing test's coverage, naming the covering test;
- a per-helper test the public surface's test already covers, naming that public-surface test;
- a review-fix test added for a finding that was not a coverage gap, naming that finding;
- a behavior the diff adds with no coverage, naming the uncovered behavior.

The bound on this new BLOCKING edge: a reviewer may raise a redundancy finding only by naming the existing test that covers the behavior, so it cannot block on a vague suspicion.
The Discussion-Review rubric is not touched.
After the edit, "independently reviewable/testable unit" and "an independently testable symbol gets its own card" are gone from the Plan-Review rubric.

`claims_test.go`, `wordingClaims`:

- `loom-rubric-plan-review.md` row: replace the `independently reviewable/testable unit` claim with a claim on "independently reviewable unit" and one on the public-surface phrase; add a `mustNot` claim for "reviewable/testable"; add a claim on `PATTERN-test-economy`.
- `loom-rubric-webster-review.md` row: add a claim on `PATTERN-test-economy`.

No new test function is added; the existing table carries the change.

**ImpactSummary:** two rubric rows of the existing claims table in `contracts/stencils`; both rubrics are read by Bouncer and Burler of their segment, and no Go production code changes.

**Commit:** 3: review rubrics: granularity by public surface, redundant tests are findings
