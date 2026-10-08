# Card 2 — discussion-plan-fans

**Edit:**
- `internal/burlerengine/template.yaml`
- `internal/burlerengine#TestConfigTemplate_DecodesAndIsSelfConsistent`

**Intent:**
Decision D3: the embedded `burler.yaml` template gains the lenses `goal-scope`, `pattern-fit`, `scenarios`, `leanness`, `decision-coverage`, `card-structure` and `verify-tests`, and the fans `discussion: [generic, goal-scope, pattern-fit, scenarios, leanness]` and `plan: [generic, decision-coverage, card-structure, verify-tests, pattern-fit]`.
Each lens text carries the emphasis D3 gives it, is phrased as emphasis, and ends with the library's "Report anything else you notice too — emphasis, never exclusion." line.
A lens that emphasizes a rubric item or a writer-stencil rule names the item by its bold title in `contracts/stencils/loom/loom-rubric-discussion-review.md`, `contracts/stencils/loom/loom-rubric-plan-review.md` or `contracts/stencils/loom/loom-template-discussion.md` and never paraphrases its rule, so the rubric stays the authority.
`leanness` is the discussion fan's lens that seeks content to move or cut, so the fan does not multiply the additive bias the rubric's relocation item counters.

`TestConfigTemplate_DecodesAndIsSelfConsistent` asserts the `discussion` and `plan` fans resolve through `ResolveFan` in the order above, that every lens a fan names exists, and that every lens text ends with the emphasis line; its lens and fan counts follow the template.

**ImpactSummary:** Template-only content plus its self-consistency test; no Go caller changes, and seeded hubs reach the new names through card 1's fallback.
