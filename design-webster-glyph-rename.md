# Webster rejects a batch whose later card renames what earlier cards glyph

`webster-glyph-rename` · Note · bug, webster, quarry

## Seen (2026-10-14, `tender`, batch `07-proc-process-listing`)

Webster rejected cards 8–10 because their `plan:internal/pausecli#…` and `plan:internal/pauseengine#…` glyphs did not resolve at HEAD.
Card 12, the plan's last card in the same batch, renamed those packages to `tendercli`/`tenderengine` on purpose, so those glyphs can never resolve against the batch's final tree.
All 21 rejected glyphs resolve under the new package names (checked by the recovery strand with `lyx quarry resolve`).
The recovery strand found the work complete at `5348618b4` and could not clear the rejection; there is no verb to accept a batch by hand.

## The bug

Webster's glyph check resolves every card's glyphs against the batch's final HEAD, so a plan whose later card renames or moves what an earlier card's glyphs name fails the check though the work is right.

## Ways out to weigh

- Resolve each card's glyphs at that card's own commit.
- Or follow renames: accept a glyph that resolves at the card's commit, or under a package the plan itself renames.
- Plan side: Plan-Review or `validate-plan` flags a rename card whose glyphs earlier cards depend on, or the Batchifier puts a rename in its own later batch.
