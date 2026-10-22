---
verdict: APPROVED
---

## Scope against the board entry

The decision record holds no `Added after Discussion` entry, so the board entry's body is the measure.
Its Idea has three points, and the diff of PR 812 at `65fb84cba` makes each, in three stencil files and nothing else:

- The Discussion writer stencil, `contracts/stencils/loom/loom-template-discussion.md`, tells the writer to research a factual question the design rests on (versions, upstream behaviour, what the code does today), record its answer with its evidence, and never defer it or hand downstream a halt-on-discovery rule.
- The Discussion-Review rubric, `contracts/stencils/loom/loom-rubric-discussion-review.md`, flags a deferred research question and a halt-on-discovery rule.
- The Plan-Review rubric, `contracts/stencils/loom/loom-rubric-plan-review.md`, flags a card that carries a research step the design depends on, or a halt rule for such a fact.

The diff changes nothing the entry does not call for.

## Open risks

The record's one open risk, the writer's Step 2 exclusions read as licence to defer, is answered in the diff by the line saying the fence excludes plan-level detail and never a fact the design rests on.
It names no unrecorded parent correction.

## Overlaps

No parent commit is listed.

- `quarry-csharp` (row 5): its D8 rewords the Plan-Review rubric's rule that an `Edit` names Go file paths, at a different place in `loom-rubric-plan-review.md` than this run's new bullet.
  Text-only overlap, no design conflict.
  Landing order: this run (own row 21) lands first, and `quarry-csharp` takes the change at Merge-In.
- `decision-add-notifies` (row 0) changes `contracts/stencils/darn/darn-template-write.md` and `contracts/stencils/orch/orch-template-role.md`, neither file this run touches; no overlap.
- `machine-handover`, `shuttle-open-agent-defaults`, `darn-verify-fixes` and `gate-tier-verbs` touch none of these stencils and record no decision on Discussion research or review flags; no overlap.

## Superseded

None.
`card-halt-is-question` concerns how a card-requested halt is routed, which this run leaves as it is.
