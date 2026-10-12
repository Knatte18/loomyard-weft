# Kick-start pack: read every card's spans in the first turn

`kickstart-read-all-first` · Task · enhancement, webster, stencil · running · Plan-Burler-Fix

The operator's intent for the kick-start pack was that the implementer fork reads everything in its first turn: every card file and every listed span for every card of the batch, all in parallel, before any edit.

`webster-pack-read-set` (landed `113593990`) built it per card instead: pack-on step 1 has the fork read the card file and the spans listed under that card in one turn, then edit, then move to the next card.
Because pack offsets are computed at begin-batch, a later card's spans can be stale once an earlier card edited the same file, so the landed text adds a rule to read such a file whole and find the member by its glyph's member name.

## What to change

- Pack-on step 1: in the first turn, read every card file of the batch and every span listed under every card, in parallel, with the Read tool's offset and limit, before any edit.
- Drop the whole-file rule for files an earlier card edited: offsets are all valid in the first turn, and later edits use the Edit tool's text matching, which does not depend on line numbers.
- The pack stays grouped per card in the prompt so the fork knows which span serves which card; only the read timing changes.
- The fork still reads further (callers, whole files) whenever a card needs it.

## Alternative (operator)

Resolve the pack fresh with quarry before each card, top down, instead of once at begin-batch.
Each card's spans then match the file as earlier cards left it, so per-card reads stay valid and the whole-file rule can go too.
This keeps the first turn's context small, at the cost of one quarry call per card.
