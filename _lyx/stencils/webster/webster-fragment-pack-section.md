<!-- This is the kick-start pack section of webster's fork prompt.
     renderPackSection (render.go) reads it through stencilstore, fills its one marker with the Go-rendered card subsections, and strips this banner and trailing newline;
     RenderForkPrompt fills the result into the optional kickstart_pack marker of webster-body-implementer.
     pack_cards holds one subsection per card of the batch, each rendered from webster-fragment-pack-card.
lyx-stencil: sha256=4a7e5967ce6f77b8abc3d8a422b3257245f68fc484e327e82a1f981e34151c6d build=1135939906fcdddd4cfdcf1bd7b46ad7ace09268 time=2026-10-12T11:39:01Z -->

## Kick-start pack

The member glyphs on each card of this batch, resolved against the tree at the batch's start.
Each card's subsection sits under a heading that is that card's path in the card list above.
Each entry gives the glyph, then one line per declaration with its file, line span and signature, and the card's unresolved refs come after its entries.

{{.pack_cards}}
