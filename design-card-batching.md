# A batchifier that groups small related cards into one Webster batch

`card-batching` · Notes · feature

Depends on: [`plan-glyph-precision`](design-plan-glyph-precision.md)

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
