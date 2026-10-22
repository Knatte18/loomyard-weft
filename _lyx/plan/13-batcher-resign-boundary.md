# Card 13 — batcher-resign-boundary

**Edit:**
- `internal/batcher#renameBoundaries`
- `internal/batcher_test#TestCostBatchifier_Limits`

**Uses:**
- `internal/batcher_test#groupCard`

**Intent:** Keep a C# re-sign and a later use of its member out of one batch (D7).

A later card names a C# member re-signed by an earlier card by its old glyph, and card 12 rewrites that glyph only when the re-sign's batch is recorded, so the two cannot share a batch.
`renameBoundaries` gains a second rule beside the cover rule: for a card holding an Edit group re-sign arrow (the group's `Resigns`) whose target `planparser.ParseRef` parses to a C# glyph, its boundary is no later than the first later card whose Targets or Uses name that target's exact string.
The rule only adds boundaries, like the cover rule, so every grouping batchifier gets it; a Go arrow adds none.

Tests: `TestCostBatchifier_Limits`' table, which shares one `go` plan today, gains a per-row `language` field, defaulting to `go`, from which each row builds its plan.
Its new rows build the re-sign card through `groupCard` with an Edit group whose `Resigns` holds the arrow and whose `Refs` holds the target, and set card 2's `Uses` or `Targets` on the card:
- under `csharp`, card 1 re-signs `Glyphchain.Sharp#Renderer.Draw(int)` and card 2 lists it under Uses: two batches under limits that would otherwise group them;
- under `csharp`, the same with card 2 listing it under Targets: two batches;
- under `go`, card 1 re-signs `internal/a#Foo` and card 2 lists it under Uses: one batch.

**ImpactSummary:** one more boundary source in the batcher's shared boundary rule; Go plans batch exactly as before.
