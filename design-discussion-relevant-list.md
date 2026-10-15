# Discussion hands Plan a list of the relevant code

`discussion-relevant-list` · Task · enhancement, loom, stencil

Depends on: [`lighter-test-rules`](design-lighter-test-rules.md)

## Seen

In the 10 runs before 2026-10-15 (`tools/tokencount/tokencount.sh -last 10`), Plan-Write was the second-largest cost after the Webster forks.
The plan lead session read 108M cached tokens, fork0 (the plan-writing fork) 51M and the five lens forks 44M.
The plan lead peaks at 130k to 370k context, and fork0 and every lens fork inherit about 80% of that peak; each message re-reads it.
Lens forks never re-read the plan (0 of 219 calls); they spot-check code briefly.
The cost comes from many messages on a large context, not from the reads themselves.

## Operator view, not a decision (2026-10-15)

The plan writer must know everything relevant before it writes.
An alternative: a separate agent between Discussion and Plan does the research and settles what is relevant.

## Orch notes (my view, not decided)

- The step writes a durable record (for example member glyphs plus short excerpts, the shape of the Webster kick-start pack), which Plan-Write reads instead of exploring, so its lead and forks start smaller.
- The research agent can run on a cheaper model, and its own exploration context is dropped when it ends.
- Risk: the plan writer re-explores anyway, or the record misses something and the plan goes wrong; measure with tokencount and plan review findings before and after.
- Related notes: `fan-master-preread`, `review-inline-code`.

## Discussion so far (2026-10-15)

- Orch: model price matters little here, since nearly all tokens are cache reads (tokencount estimates about $2.7 per weighted million for the Opus plan lead against $2.0 for the Sonnet Webster forks); the gain comes from a smaller context, and a separate research agent adds a handoff that may lose understanding.
- Orch proposal, not decided: no new role; the Discussion writer, which already reads the relevant code, writes a separate file beside its discussion (for example `relevant.md`) listing the files and members it found relevant, in plain code names with one line of why each, not glyphs; the discussion prose stays free of glyphs.
  Plan-Write starts by reading that list and may read more.
  Discussion-Review does not review the list as design, so the review does not grow.
  Measure the plan lead peak context and the Plan cost with tokencount over a few runs before and after.
- Operator view, not a decision: the Discussion agent has already looked at everything relevant, so the list records what it read and found relevant, with no extra search for Plan.
- Operator: discuss further before this becomes a task.

## Operator decision 2026-10-15

- No new role: the Discussion step is extended.
- The Discussion writer writes a separate file beside the decision record and the support log, `relevant.md`, listing the files and members it read and found relevant, in plain code names with one line of why each; not glyphs, and no extra search for Plan.
- The discussion prose stays free of glyphs.
- Plan-Write starts by reading the list and may read more.
- Discussion-Review does not review the list as design.
- The new file goes into `contracts/specs/loom-discussion-spec.md`, which pins the files a design step writes, together with the Discussion and Plan stencils (and `loom-plan-spec.md` where it names what Plan reads).
- Measure the plan lead peak context and the Plan cost with `tools/tokencount/tokencount.sh` over a few runs before and after.