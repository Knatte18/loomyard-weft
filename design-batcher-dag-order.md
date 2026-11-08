# Batcher groups cards by the dependency graph, not only contiguous runs of card order

`batcher-dag-order` · Note · enhancement, webster

## Seen (operator and orch, 2026-11-08)

- The cost batcher (`internal/batcher/cost.go`) splits the plan's cards into the fewest contiguous segments of the card order that fit the context budget, so two cards can share a batch only when every card between them does too.
- Webster's own check does not need that: `CheckBatchOrder` (`internal/websterengine/sequence.go`) refuses only an edge from a batch to an earlier one, with edges derived from the cards' Targets and Uses refs, and two cards writing one Targets entry ordered by card number.
- So with cards 1, 2 and 3, where 3 uses 1's target and 2 shares nothing with either, cards 1 and 3 could run as one batch and 2 after it, but the batcher never forms that.

## Idea (operator, 2026-11-08)

- The batcher groups by the card dependency graph, not by card order: any grouping whose batch order keeps every edge forward is allowed, and the batcher picks the one with the fewest batches that fit the budget, preferring cards that read the same files in one batch, since that lowers the batch's peak context.
- The batch-order check, the digest of the batch before, and the partition record stay as they are; only the batcher's grouping changes.
- What it saves is unmeasured: fewer batches mean fewer forks and fewer verify rounds, but how many plans have such gaps is not known.
