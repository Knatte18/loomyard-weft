# Rename the Merge-In row to Parent-Align

`merge-in-parent-align` · Note · enhancement, loom, shed, stencil

## Seen (2026-10-24)

Since `loom-tail-review` landed, loom's `Merge-In` row does more than merge: it takes the parent in, and when the parent brought commits in, or the merge conflicts or fails its gate, its fixer resolves the conflicts, makes the tests green and checks that the task's work still matches its decision record against what came in, escalating a decision that collides with landed work.
Without new parent commits and with a green gate it runs no fixer.
That second part is the old alignment check, moved in and narrowed to what the parent brought.
`Finalize` keeps its own small merge-in under the parent lock, as a last sync.

## Idea (operator, 2026-10-24)

Rename the row `Merge-In` to `Parent-Align`, so its name says it aligns the task with its parent, not only merges.
The rename covers the recipe row in loom and darn, the producer and engine names, the `merge_*` config keys it reads, the stencils and docs.
Row names are durable resume identities, so a run parked on `Merge-In` is restarted, as the recipe already says for retired rows.
