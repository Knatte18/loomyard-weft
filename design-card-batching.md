# A batchifier that groups small related cards into one Webster batch

`card-batching` · Note · feature, webster

## Why

`internal/batcher` has a registry but only the `identity` batchifier, so every card is its own Webster batch.
On `test-suite-change-amplification` (45 cards, 2026-10-03) each batch carried a fixed overhead besides the fork's own work: `record-batch`, `begin-batch`, three short Merriam turns and the fork's start-up, about 16–18 s per batch after quarry v0.2.1 (over a minute before it, #345), plus a fork prompt and Merriam polling per batch.
Many cards there take 40–120 s of work, so for them the overhead is a large share.

## Want

A mechanical batchifier (operator): it groups cards from the plan's own card fields (`Create`, `Edit`, `Uses`, renames), never from an LLM's judgment, selected through `batcher.yaml`'s `active` key, the identity batchifier staying the default.
Cards whose fields show no dependency on each other and no shared target can share a fork; a card that `Uses` another's `Create` follows it.

## Waits on `plan-glyph-precision`

Plans list file paths, not member glyphs, so card fields today only support file-level grouping, which is too coarse to be the point.
The batchifier is built once cards name the members they touch.

## Trade-offs to settle

- A larger batch gives the fork a larger context and makes a failure harder to localize to one card.
- The per-batch validation and drift check (`begin-batch`, `record-batch`) then runs per group, not per card; the operator wants plan drift caught as it happens.
- It should fit the parallel-execution direction in `manifest/designs/webster-parallel-execution.md`, where the same DAG decides which groups may run concurrently.

Member-level grouping depends on `plan-glyph-precision`; the quick fix below does not.

## Quick fix for tokens (operator, 2026-10-07)

The operator wants better batches now, as a quick fix for token use, not one batch per card, and chose it as the first token saving to build.
A token count over the five latest runs (`tools/tokencount`, 2026-10-07) put Webster's implementer forks at about 54% of all weighted tokens, nearly all of it cache reads: each fork reads a large context afresh (574M cache-read tokens over 81 forks; the two test-suite prunes ran 26 and 18).
For tokens, file-level grouping is enough: cards that edit the same files or package and follow each other in plan order share one fork, so their shared context is read once, with a cap on cards per batch to keep the context and failure localisation in bounds.
So this quick fix does not wait on `plan-glyph-precision`; finer member-level grouping can follow once cards name members.

A fork's size is not the master's cost: in the tests while Webster was built and hardened, a fork that succeeded added only about 1,000 tokens to the master (operator, 2026-10-07).
The cost is in the forks themselves: a sample of five forks on `bugfix-webster` started at 56–89K context tokens and grew to 155–217K over 80–140 messages, each message reading the whole context again.
Fewer forks with better batches means less repeated exploration and far fewer fork start-ups, so this is where the token saving starts.

## Design so far (operator, 2026-10-07)

A Go batchifier, never an LLM agent.
It cannot merge cards blindly: on 2026-10-07 single cards cost a Webster fork hundreds of thousands of tokens, and such a card must never share a fork.

- **Size each card mechanically**, from what the card and the disk say before the run: how many files it creates, edits or deletes, the line count of the files it edits, its test files and their size, and its `Uses` entries. A fork spends its tokens reading files and running tests, so these are the signals.
- **A card above a size threshold always runs alone.**
- **Small cards group** only when they follow each other in plan order (so no dependency is broken), share files or a package (so the shared context is read once), and stay within a size budget and a cap on cards per batch.
- **Calibrate** the estimate against real use: `tools/tokencount` extended to report tokens per fork, one fork being one Webster batch, compared with each batch's estimate to tune the weights. Start cautious, with a low budget and a cap of three or four cards.

## Profiles in `batcher.yaml`

Several named batchifier profiles, all configured in `batcher.yaml`, with `active` naming the one in use, so the operator switches between them by changing one key (operator, 2026-10-07).
Not a single batchifier with one setting: each profile carries its own threshold, budget, card cap and weights, and the identity batchifier (one card, one batch) stays available as a profile of its own.
