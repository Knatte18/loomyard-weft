<!-- This is one card's subsection of the kick-start pack in webster's fork prompt.
     renderCardPack (render.go) reads it through stencilstore, fills its markers, and strips this banner and trailing newline;
     renderPackSection joins one such subsection per card and fills the result into the pack_cards marker of webster-fragment-pack-section.
     card_path is the card's path in the card list, pack_entries lists the card's member glyphs each with its declarations,
     and pack_unresolved is optional: the card's unresolved list when it has unresolved refs, and nothing otherwise.
lyx-stencil: sha256=cfa62a5bf050fde4db35f25cf386e132928c4b4d327753a2177857307ac895da build=1135939906fcdddd4cfdcf1bd7b46ad7ace09268 time=2026-10-12T11:39:01Z -->

### `{{.card_path}}`

{{.pack_entries}}{{.pack_unresolved}}
