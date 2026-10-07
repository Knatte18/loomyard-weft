# A batchifier that groups small related cards into one Webster batch

`card-batching` · Task · feature, webster · running · Discussion-Bouncer

Depends on: [`bugfix-webster`](design-bugfix-webster.md)

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

## Deciding a batch from a cost model (proposed by the hub, accepted by the operator, 2026-10-07)

Grouping cards that merely follow each other and belong together is a start, not the rule (operator).
The batchifier decides from an estimated cost instead:

- **Cost model.** A fork costs roughly its messages times its context size, since every message reads the whole context again.
  Each card gets an estimate of its messages and of how much context it adds, from the signals above.
  Running card B in card A's fork saves B's fork start-up and the files they share, which are read once; it costs B carrying A's accumulated context in every message.
  So merging pays only when the cards share much and both are small, and the model, not a judgment, says when.
- **Overlap** is measured between the cards' read sets: the files they edit or use, their package and their test package.
  Two cards editing the same file share more than two cards in the same package.
- **Hard limits** hold whatever the model says: plan order and dependencies are never broken, a card above the threshold runs alone, and a batch stays within its card cap and budget.
- **Optimal split.** Plan order is a valid order, so the batchifier splits the card sequence into contiguous segments with the lowest total estimated cost: a small dynamic program over segments, exact and testable, rather than greedy merging.
  Moving cards next to each other where the dependencies allow, so cards of one package separated by an unrelated card can join, is a later step.

## Testing

- **The batchifier itself** is a pure function from cards and file stats to batches, so unit tests pin the hard limits on every input: order and dependencies kept, an oversized card alone, caps and budget kept.
- **The estimator offline**: past runs ran the identity batchifier, one fork per card, so `tools/tokencount` extended to tokens per fork gives a measured cost for every past card.
  Running the estimator on those plans and comparing with the measured cost calibrates the weights without running anything new.
- **The saving itself** cannot be measured offline, since a merged batch's cost was never observed.
  It is measured on real runs, switching profiles in `batcher.yaml` and comparing tokens per card against the identity profile's history, starting cautious.

## First step: calibration against past plans (operator, 2026-10-07)

The first cards of this task build the calibration, before any grouping:

- **Tokens per fork, tied to its card.** `tools/tokencount` reports each Webster fork's usage, and ties the fork to the card it ran, through the card name in the fork's prompt or the `begin-batch` and `record-batch` commits' times.
- **The past plans.** Every past run's plan is in the weft history (the `loom: plan artifacts for <slug>` commits), with the cards' `Create`, `Edit` and `Uses`.
- **The disk as it was.** File and test sizes come from the run's base commit on main, not today's tree.
- **The comparison.** The estimator runs on each past plan and its estimate per card is set against the card's measured tokens, to tune the weights before a profile groups anything.

`tools/` follows PATTERN.md and the repository's enforcement scans like every other package.

## Profiles in `batcher.yaml`

Several named batchifier profiles, all configured in `batcher.yaml`, with `active` naming the one in use, so the operator switches between them by changing one key (operator, 2026-10-07).
Not a single batchifier with one setting: each profile carries its own threshold, budget, card cap and weights, and the identity batchifier (one card, one batch) stays available as a profile of its own.
