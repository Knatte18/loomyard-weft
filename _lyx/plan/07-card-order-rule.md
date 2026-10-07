# Card 7 — State that a plan runs in card order

**Prosa:**
- `contracts/specs/loom-plan-spec.md`
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Intent:**
Implement D12.
Rewrite `loom-plan-spec.md`'s "Plan vs. schedule" section: a valid plan is executable strictly in card-number order, a card never depends on anything a later card does, and Webster runs the cards in that order as written and never sorts them.
Drop the "DAG of intent" and "not itself an execution order" wording and the forward-looking executor paragraph built on it; keep the note that `uses-later-target` and `delete-before-reference` refuse the ref-matched cases.
In `loom-template-plan.md`, where it describes what a card is and the Card Index, tell the writer that the plan runs in card order and point at the spec section rather than restating it (`PATTERN-producer-pointer-rule`).
In `loom-rubric-plan-review.md`, add a finding under "Also flag": a card-order violation of that spec rule, pointing at the section and listing no cases.
Semantic line breaks throughout.
