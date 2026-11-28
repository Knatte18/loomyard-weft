# Weaver runs each batch as a Shed row of the new kind and drops its own stop, re-entry, resume and lock code

`weaver-onto-shed-rows` · Note · enhancement, weaver, shed

Depends on: [`shed-dynamic-rows`](design-shed-dynamic-rows.md)

## Seen (operator, 2026-11-28)

About 15–20% of Weaver's production code repeats in concept what Shed does: its stop and re-entry glue, part of its resume and its run lock.
It exists because Shed had no row that repeats or unfolds from a list.

## Idea

Once Shed has that row kind and it has proven itself, move Weaver onto it:
each batch becomes a pass or an unfolded row with Weaver as a thin producer,
and Weaver's own stop record, re-entry, in-row resume and run lock give way to Shed's.
The model ladder, batch partitioning, verify fixer, in-flight strand probing and reset stay Weaver's, since Shed has no counterpart for them.

## Bounds

- Weaver's behaviour seen from the run stays the same: the same batches, ladder, journal, summary and outcome.
- A bench against today's Weaver on the same plans checks it before Weaver's own loop is removed.
