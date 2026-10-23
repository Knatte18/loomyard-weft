# Card 10 — Extends guidance for plan writer and reviewer

**Prosa:**
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils/loom/loom-template-plan.md`

**Intent:**
D6: plan review checks the label, and the writer is told when to use it.

`loom-rubric-plan-review.md`'s "Also flag" list gains a bullet: an `Extends` on a ref the card rewrites, reorders or removes content of is a finding, as a missing `Uses:` is; the finding names the ref and moves it under `**Edit:**`.
Its wording keeps the writer/reviewer symmetry: it flags only what the plan-writer stencil tells the writer to do.

`loom-template-plan.md`'s card-field section adds `**Extends:**` to the list of type labels and says when to use it: a ref that already exists and that the card only adds to (a flag, a list line, a doc paragraph, a spec entry, a verb in a wire function) without changing anything already there; a re-sign arrow never goes under it; two Extends on one ref make no dependency, unless both cards fence a line for one PATTERN entry ID.
It also says the card that writes the code writes its docs, naming the doc files under `**Extends:**`, so no separate closing docs card is planned.

The sentences that list or count the other labels name `Extends` too: the stencil's "`**Custom:**` is a last resort, used only where none of the other six genuinely fits" and its next sentence's "a multi-label combination of the other six" each become "the other seven", the rubric's `Custom` bullet's list of `Create`, `Edit`, `Delete`, `Rename`, `Move` and `Prosa` gains `Extends`, and the rubric's "a multi-label combination of the other six" becomes "the other seven".

The rubric and the stencil are prose read by agents; no test asserts their text.
