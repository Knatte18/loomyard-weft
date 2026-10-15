# Card 2 — inline-fragments

**Create:**
- `contracts/stencils/webster/webster-fragment-fresh-read-pack-inline.md`
- `contracts/stencils/webster/webster-fragment-locate-pack-inline.md`
- `contracts/stencils/webster/webster-fragment-pack-section-inline.md`
- `plan:contracts/stencils#WebsterFragmentFreshReadPackInline` -> `var WebsterFragmentFreshReadPackInline []byte`
- `plan:contracts/stencils#WebsterFragmentLocatePackInline` -> `var WebsterFragmentLocatePackInline []byte`
- `plan:contracts/stencils#WebsterFragmentPackSectionInline` -> `var WebsterFragmentPackSectionInline []byte`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#WebsterFragmentLocatePack`
- `contracts/stencils#WebsterFragmentFreshReadPack`
- `contracts/stencils/webster/webster-fragment-locate-pack.md`
- `.gitattributes`
- `contracts/stencils/webster/webster-fragment-pack-card.md`
- `contracts/stencils/webster/webster-fragment-pack-empty.md`
- `contracts/stencils/webster/webster-fragment-pack-unresolved.md`
- `contracts/stencils#wordingClaims`
- `contracts/stencils#TestStencils_TopLevelMarkers`

**Intent:** Add the three frozen inline-layout fragments of D2 and D3 and register them, so card 3 can render the inline layout from stencils (`PATTERN-stencil-ownership`).
No renderer reads them yet, so this card changes no prompt.

Each new file is a banner comment, a blank line, then a body equal byte for byte to the matching pack-on fragment's body at `1e39afc10^`, banner stripped; take each body from `git show 1e39afc10^:contracts/stencils/webster/<fragment>.md`:

- `webster-fragment-fresh-read-pack-inline.md` takes the body of `webster-fragment-fresh-read-pack.md` at `1e39afc10^`: the one-turn read instruction ("Your first turn, before any edit, reads in parallel every card file the card list below points at …" through "… so it is never located.") and the Edit-only rule. No marker.
- `webster-fragment-locate-pack-inline.md` takes the body of `webster-fragment-locate-pack.md` at `1e39afc10^` ("   The kick-start pack below lists, under each card's path, …"). One marker, `specs_dir`.
- `webster-fragment-pack-section-inline.md` takes the body of `webster-fragment-pack-section.md` at `1e39afc10^`: `## Kick-start pack`, the three framing lines ending "… unresolved refs come after its entries." with "the card list above", then `{{.pack_cards}}`. One marker, `pack_cards`; no read instruction and no "go back to your prompt" line.

Each banner names the fragment's role in the inline layout: which marker of `webster-body-implementer` it fills (`fresh_read`, `locate`, or the new optional `kickstart_pack_inline` that card 3 adds) when begin-batch built a pack and `kickstart_pack_file` is `false`, and that it is the measurement baseline: its text is the `1e39afc10^` text and changes only when the layout comparison itself is meant to change, never to mirror an edit to the file-layout fragments.
The banners of `webster-fragment-pack-card.md`, `-pack-empty.md` and `-pack-unresolved.md` change to say the subsection goes into the kick-start pack file or the fork prompt's inline pack section, since both layouts share them; their bodies do not change.
The `-pack-empty.md` banner keeps clear of the phrase "this batch's cards", which its existing claim row refuses anywhere in the file.
The doc comments of `WebsterFragmentLocatePack` and `WebsterFragmentFreshReadPack`, and the banner of `webster-fragment-locate-pack.md`, say they fill their marker when the fork prompt carries a kick-start pack in the file layout, since card 3 fills the `-inline` siblings in the inline layout; their bodies do not change.

Registration, as `1e39afc10` did for `webster-fragment-pack-read`:

- `contracts/stencils/stencils.go` gains the three `//go:embed webster/<file>` vars, each after its file-layout sibling, each with a doc comment naming the marker it fills in the inline layout.
- `entries` gains one row per new stencil, each after its file-layout sibling; `TestRegistry_MatchesOnDiskTree` pins that every file has a row.
- `.gitattributes` gains a `text eol=lf` line per new file, after its sibling's line.

Tests:

- `TestStencils_TopLevelMarkers` gains `{"webster-fragment-fresh-read-pack-inline", WebsterFragmentFreshReadPackInline, nil}`, `{"webster-fragment-locate-pack-inline", WebsterFragmentLocatePackInline, []string{"specs_dir"}}` and `{"webster-fragment-pack-section-inline", WebsterFragmentPackSectionInline, []string{"pack_cards"}}` (acceptance: each fragment renders with its markers resolved).
- `wordingClaims` gains one row per new file:
  fresh-read inline `wantAll` "Your first turn, before any edit, reads in parallel every card file the card list below points at and every span the kick-start pack lists under every card", "A span listed under several cards is read once." and the Edit-only sentence;
  locate inline `wantAll` "   The kick-start pack below lists, under each card's path, where each member glyph on that card is, and your first turn read every listed span." and "`{{.specs_dir}}/loom/loom-plan-spec.md`";
  section inline `wantAll` "## Kick-start pack\n\nThe member glyphs on each card of this batch, resolved against the tree at the batch's start." and "that card's path in the card list above", and `wantNone` "Read all of this in one turn" and "Then go back to your prompt".
  No row refuses the phrase "kick-start pack file", since a claim scans the banner too and a banner may contrast the two layouts.
  The exact equality of each rendered body with the `1e39afc10^` text is pinned by card 3's render row.

**ImpactSummary:** Three new embedded stencils with registry, `.gitattributes`, marker and claim rows; four existing fragments get banner-only edits and two embed vars doc-comment edits, which `stencil.StripLeadingComment` removes before render, so no rendered prompt changes.

**Commit:** 2: webster inline-layout pack fragments
