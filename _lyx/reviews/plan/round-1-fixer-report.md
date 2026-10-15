# Fixer report — plan, round 1

## Changes

- **F1** (LOW, scope), fixed as proposed: card 3's `BeginBatch` bullet under "Render (D4)" in `_lyx/plan/03-render-layout.md` now says its doc comment changes so the pack file is written beside the prompt only when the batch has a pack and `kickstart_pack_file` is true, the file layout.
  The premise held: `internal/websterengine/beginbatch.go:427` says the pack file is written whenever the batch has a pack.
  The `**Edit:**` list is unchanged, since it already names `internal/websterengine#BeginBatch`.

## Disputed

None.

## Deferred

None.
