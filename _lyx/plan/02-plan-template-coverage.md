# Card 2 — plan-template-coverage

**Prosa:**
- `contracts/stencils/loom/loom-template-plan.md`

**Edit:**
- `contracts/stencils/claims_test.go`

**Intent:** Reword the plan writer's stencil so it asks for coverage, not a bundled test file (D2, D4 and the Step 0 note), and move the claims that pin its old wording onto the new wording (D9).

`loom-template-plan.md`:

- Step 0's rationale: "the card-granularity rule and the bundle-your-own-test rule are testing judgments" names the test-coverage rule instead of "the bundle-your-own-test rule".
- "What a card is", item 3: replace "**Bundles its own test when it introduces new behavior** — … `verify:` commands are not a substitute for a bundled test;" with a bold criterion on coverage, worded as D2 states it: a card that introduces behavior carries the coverage for that behavior in the same card, extending an existing test where one covers the surface and adding a new test only for behavior no existing test covers, per `PATTERN-test-economy`.
  Drop the "structuring the change so it is testable (extract a helper …)" example: a helper is tested through its public caller.
  Keep that `verify:` commands are no substitute for test coverage, and that only pure refactors and renames rely on existing tests.
- The sentence "An implementation card that bundles its own new test file writes `**Edit:**` … — this is the normal shape for such a card, not an exception." becomes: the normal shape extends an existing test file under `**Edit:**`, and a `**Create:**` test file, after the implementation's `**Edit:**`, appears only for a surface no existing test file covers.

The stencil gains no granularity rule (D3 leaves it to the spec and the Plan-Review rubric).
After the edit, none of these phrases remain: "Bundles its own test when it introduces new behavior", "so it is testable", "not a substitute for a bundled test", "the bundle-your-own-test rule", "this is the normal shape for such a card".

`claims_test.go`, the `loom-template-plan.md` row of `wordingClaims`: in the `wantAll("the card-granularity contract reaches the agent", …)` group, replace "Bundles its own test" and "not a substitute for a bundled test" with distinctive phrases of the new item 3 (one naming `PATTERN-test-economy`, one carrying the no-substitute clause).
Join a `wantNone` group to the same row barring "Bundles its own test" and "not a substitute for a bundled test", so the old wording cannot return.
No new test function is added; the existing table carries the change.

**ImpactSummary:** one stencil read by the Plan-Write producer and one row of the existing claims table in `contracts/stencils`; no Go production code changes.

**Commit:** 2: plan stencil: cards carry coverage, not a bundled test file
