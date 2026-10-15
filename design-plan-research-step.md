# A research step between Discussion and Plan that settles what is relevant

`plan-research-step` · Note · enhancement, loom, stencil

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