# Card 12 — plan-review-rework-answer-key

**Prosa:**
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/recipes/loom-recipe.yaml`

**Edit:**
- `contracts/stencils/rubric_test.go`

**Intent:**
Make the live generation's findings part of Plan-Review's answer key in a rework generation (D9), so a rework card is not flagged as unlicensed scope and finding coverage is checked.

Both the Plan-Burler row's `fasit.instructions` in `loom-recipe.yaml` and `loom-rubric-plan-review.md` state:

- how to find the live generation's round: among `_lyx/loom/rework/round-<N>/` directories, the highest `N` whose `record.json` `first_card` equals the `first_card` in `_lyx/plan/00-overview.md`'s frontmatter (absent means `1`);
- with such a round, its `findings.md` joins the decision record as the answer key: every finding is covered by some card, and every card's scope is licensed by the decision record or a finding;
- with no such round (generation 0, or a fresh Plan-Write plan after an operator `goto Plan-Write`), the decision record alone is the answer key, as today.

The rubric's "Fidelity to the decision record" item is reworded to the answer key, and the round's `prior-generation/` archive is named as context that is never the subject of a finding.

Tests in `rubric_test.go`: the plan rubric names `findings.md`, the live-generation round rule (`first_card`), and the generation-0 fallback to the decision record alone.

**ImpactSummary:** stencil-test assertions only; the rubric and recipe text change what the Plan-Review agents read, and no Go code reads either.
