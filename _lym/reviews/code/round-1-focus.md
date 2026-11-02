---
round: 1
focus:
  - "contracts/stencils/loom/loom-template-discussion.md against loom-template-plan.md: does the Discussion-Write guidance match the Plan guidance in form (batch in one message, whole question per prompt, short answer naming files and spans, optional not commanded, no fork)?"
  - "The subagent model key (internal/loomengine/config.go, template.yaml): does it sit beside the writer-model keys, default to sonnet, and name in its template comment which stencils read it?"
  - "Model wiring: does each of the Plan, Discussion-Write and darn-write stencils take the model from the new key through loomengine/plan.go, discussiontable.go, loomcli/wiring.go, loomcli/darnwiring.go and darnengine/spec.go, with no hardcoded model: \"sonnet\" left in any stencil?"
  - "Does the darn path read the key from loom config without a layering or import breach, and does an unset or empty key fall back to sonnet rather than render an empty model?"
  - "Test economy in the changed *_test.go files: does any new assertion pin a stencil's static prose, or does any test duplicate coverage already in config_test.go or the table tests?"
  - "docs/overview.md and the repository's constraints document: does the diff land every doc the rules name for a config-key addition?"
---

Seed focus from the decision record's Design items 1 and 2 and the diff stat over the review range.
