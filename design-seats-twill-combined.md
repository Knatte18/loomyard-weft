# Discussion chair drafts with advisors, then forks itself for twill lens reviews

`seats-twill-combined` · Note · enhancement, loom, stencil, config

## Seen

`loom.yaml` `discussion_producer` takes one of `single`, `seats` and `twill`, so a run gets either a chair with advisors or a writer that forks itself into lens reviews, never both.
The chair stencil (`contracts/stencils/loom/loom-template-discussion-chair.md`) carries no twill directive.

## Idea (operator, 2026-10-11)

- The chair writes the first draft with its advisors, as seats does.
- It then forks itself once per lens of the `twill-discussion` fan, as twill does, and folds the lens findings into the record.
- The forks share the chair's prompt cache, so the lens pass stays cheap; the advisors bring an outside view the forks cannot, since forks share the chair's priors.

## Open

- Whether this is a fourth `discussion_producer` value or twill becomes a flag on top of `single` and `seats`.
- Bench it against the arms of the twill bench and the chair-blind-questions bench (`.scratch/twill-bench/results.md` holds the earlier arms while it lasts).
