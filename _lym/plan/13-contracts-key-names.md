# Card 13 — contracts-key-names

**Prosa:**
- `contracts/orchrole/orch-role.md`
- `contracts/specs/llm-model-spec.md`
- `contracts/specs/raddle-proposal-spec.md`
- `contracts/specs/shed-recipe-spec.md`
- `contracts/specs/loom-parts-spec.md`
- `contracts/recipes/raddle-recipe.yaml`
- `contracts/recipes/hardener-recipe.yaml`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/loom/loom-fragment-parts-light.md`
- `contracts/stencils/loom/loom-fragment-parts-medium.md`
- `contracts/stencils/loom/loom-template-parts-chair.md`
- `contracts/stencils/loom/loom-template-parts.md`
- `contracts/stencils/loom/loom-template-part-writer.md`
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-template-discussion-advisor.md`
- `contracts/stencils/loom/loom-template-discussion-chair.md`
- `contracts/stencils/landing/landing-template-fixer.md`
- `contracts/stencils/landing/landing-block-pr-review-operator.md`
- `contracts/stencils/drift/drift-template-expect.md`
- `contracts/stencils/weaver/weaver-body-verify-fix.md`
- `contracts/stencils/gatefix/gatefix-body-plan.md`

**Intent:** Every stencil, spec, recipe comment and the orch role under `contracts/` names a config key by its full dotted name (D3): `loom.code.max_bounces`, `tapestry.plan.reviewer_fan`, `loom.review.judge`, `drift.judge_ask_parent`, `beam.pull_request`.
The listed files are those a search for flat loom and tapestry names and bare module keys found; the writer greps `contracts/` again for each moved key's old name and each bare key, and every further file holding one is part of this card.
A command a text prints keeps the CLI's form, `lym config loom --set code.max_bounces=2`.
No stencil marker, value Go fills, or fragment Go chooses changes, so no stencil test changes; a stencil's static prose takes no test.
The recipes whose setting references card 4 moved keep them; only comments change here.
