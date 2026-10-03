# A batchifier that groups small related cards into one Webster batch

`card-batching` · Someday · feature

## Why

`internal/batcher` has a registry but only the `identity` batchifier, so every card is its own Webster batch.
On `test-suite-change-amplification` (45 cards, 2026-10-03) each batch carried a fixed overhead besides the fork's own work: `record-batch`, `begin-batch`, three short Merriam turns and the fork's start-up, about 16–18 s per batch after quarry v0.2.1 (over a minute before it, #345), plus a fork prompt and Merriam polling per batch.
Many cards there take 40–120 s of work, so for them the overhead is a large share.

## Want

A batchifier that groups consecutive cards into one fork when they are small and related (same package, or a kit card followed by its first consumer), selected through `batcher.yaml`'s `active` key, the identity batchifier staying the default.

## Trade-offs to settle

- A larger batch gives the fork a larger context and makes a failure harder to localize to one card.
- The per-batch validation and drift check (`begin-batch`, `record-batch`) then runs per group, not per card; the operator wants plan drift caught as it happens.
- It should fit the parallel-execution direction in `manifest/designs/webster-parallel-execution.md`, where the same DAG decides which groups may run concurrently.
