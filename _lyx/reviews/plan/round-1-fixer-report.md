# Fixer report — plan round 1

## Changes

- **F1** (fixed as proposed): in card 01's Intent, `doc.go` paragraph, "The three sentences from" is now "The four sentences from".
  `internal/websterengine/doc.go:156-159` holds four sentences in that span, so line 158 ("That bounds the fork's reading …") is replaced too.
- **F2** (fixed as proposed): in card 01's Intent, `webster-fragment-locate-pack.md` paragraph, the new first line and the new line-number line now sit in one fenced `text` block, in order, each with its 3-space indent, with `lyx quarry resolve` in backticks as the stencil holds it.
- **F3** (fixed as proposed, taking the fix key's second option): in the `TestRenderPrompts` multipart row bullet, `contains` now also drops the prefix "The kick-start pack below lists, under each card's path, where each member glyph on that card is", which the full new first line holds.
  It no longer gains "Your first turn, before any edit, reads in parallel every card file".
  The `inOrder` needle is now that full phrase in place of "Your first turn, before any edit", so one needle pins both presence and order.

## Disputed

None.

## Deferred

None.
