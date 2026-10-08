# Card 4 — carry-over-consumers

**Edit:**
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-template-rework.md`
- `contracts/stencils/landing/landing-template-describe.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils/claims_test.go`

**Intent:**
Tell each consumer of a carry-over entry what to do with it (D2 consumers, and the added Discussion decision on consumers per segment).

- `loom-template-plan.md`, in Step 1 beside reading the decision record: an entry between `<!-- lyx:carry-over … -->` marker lines in `## Open risks` lists review findings whose fix in the record no fresh reviewer has seen; check each against the code while exploring, and never plan an entry as scope.
- `loom-template-rework.md`, where it reads the decision record: the same sentence for a rework generation.
- `landing-template-describe.md`, under what to read and what to write: each finding a Plan-Review or Webster-Review carry-over entry lists becomes one item in the description's things to check by hand, worded as the finding and its location, never as a review round, which the stencil's no-pipeline rule already forbids naming.
  A Discussion-Review entry is not carried into the description.
- `loom-rubric-plan-review.md`, in its answer-key bullet: a carry-over entry in `## Open risks` is not part of the answer key, and it licenses no card's scope.

The Webster-Review rubric is unchanged, since it holds the code against the plan and never reads the record.
Stencil text carries no conditional and no new marker, so every existing fill stays valid.

`claims_test.go` gains one claim per stencil pinning its sentence: the plan and rework stencils' "never plan … as scope" wording, the describe stencil's "item to check by hand" wording for Plan-Review and Webster-Review entries, and the rubric's "not part of the answer key" wording.

**ImpactSummary:** Prompt text only, in four stencils read at call time; the claims table pins the new sentences and no Go behavior changes.
