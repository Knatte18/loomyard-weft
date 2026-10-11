# Cold review of the merge from the parent before PR-Gate

`merge-review` · Note · enhancement, loom, burler

## Seen (2026-10-11)

At PR-Gate the orch forks a reviewer over the whole diff, but Webster-Burler has already reviewed the task's code.
What no review has seen is the interaction with what landed on the parent while the task ran: Publish merges the parent in, and only verify runs over the result.

## Idea (operator, 2026-10-11)

A cold-start agent reviews the merge result after the parent is merged in, inside loom, and only when the merge brought in changes.
The PR-Gate fork then narrows to what needs the orch's context: the run against its board goal and the operator's decisions.

## Open

- Whether the review covers only changes from the parent that touch the task's packages or glyphs, or the whole merge.
- Where in the recipe it runs (after Publish's merge, before PR-Gate) and what a finding sends back to.
- Relation to `prgate-stale-base`: a run sent back to Publish because the parent moved during PR-Gate redoes the merge this review would cover.
- For the operator (2026-10-11, undecided): when to start this and `alignment-agent`, now that `burler-split-budget` has landed; the operator wants to discuss it later.
