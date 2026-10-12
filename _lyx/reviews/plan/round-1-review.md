---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/02-pack-per-card.md:108
    summary: Card 2 rewords the empty-pack line but leaves the multipart TestRenderPrompts row's notContains needle on the old wording, so that guard passes vacuously after the card
    fix: In 02-pack-per-card.md, in the Tests paragraph's multipart-row bullet (line 108), add that the row's notContains needle `no member glyphs on this batch's cards` (template_test.go, the multipart row's notContains) becomes `no member glyphs on this card`, so a card that names member glyphs is still asserted not to render the per-card empty line.
  - id: F2
    severity: NIT
    class: consistency
    location: _lyx/plan/03-pack-on-read-texts.md:39
    summary: Card 3 puts the whole-file read of a file an earlier card edited after step 1's two kept sentences, while D3 says only the read sentence changes
    fix: In 03-pack-on-read-texts.md, in the `webster-fragment-card-read-pack` bullet (line 39), move the clause on a span in a file an earlier card of this batch edited (read whole in the same turn, member found by its glyph's member name, signature only a cross-check) to sit right after the parallel-read sentence and before the two verbatim sentences, so the fragment is the new read text followed by step 1's two kept sentences unchanged, as D3 states.
---

### [LOW] Stale empty-line needle in the multipart render row

**Location:** `_lyx/plan/02-pack-per-card.md:108`

**Issue:** Today's multipart row in `TestRenderPrompts` (`internal/websterengine/template_test.go:805`) asserts `notContains` `no member glyphs on this batch's cards`.
Card 2 rewords `webster-fragment-pack-empty` to `no member glyphs on this card` and rewrites that row's `inOrder` and its `#### Unresolved` exclusion, but says nothing about the `notContains` needle.
Left as is, the needle can never match after card 2, so a regression where a card that names member glyphs renders the empty line passes the row.
Card 3 handles the same kind of stale needle for the locate opening; card 2 misses this one.
`TestRenderPrompts` is already on card 2's `Edit:` list, so no target changes.

### [NIT] Edited-file rule placed after the kept step-1 sentences

**Location:** `_lyx/plan/03-pack-on-read-texts.md:39`

**Issue:** D3 says the pack-on fragment keeps step 1's two following sentences verbatim, "so only the read sentence changes", and its second paragraph is part of that read instruction.
Card 3 orders the fragment as read sentence, the two kept sentences, then the edited-file whole-read clause.
That splits the read instruction across the kept sentences, and the fragment then no longer ends on step 1's text the way D3 describes.

## Rejected

- Card 2's re-sign arrows: every member whose `Intent` changes parameters or results carries one (`KickstartPack`, `index.KickstartPack`, `selectPackRefs`, `buildKickstartPack`, `RenderForkPrompt`, `renderLocate`, `renderPackSection`, `fakeIndex.KickstartPack`, `renderFork`).
  `Index.KickstartPack` is an interface method, and `Pack` and `fakeIndex` are struct changes, so none of them takes an arrow.
  `BeginBatch`, `RenderRecoveryPrompt` and `renderCardPointers` keep their signatures.
  They are listed as callers of re-signed members, which the plan template requires.
- Card 2 as one unit: it cannot be split.
  The `Index.KickstartPack` result type change does not compile in `websterengine` until `buildKickstartPack`, `RenderForkPrompt`, `renderPackSection` and `fakeIndex` take the slice.
  Any split would need a transitional shim.
- Card order: card 3 depends only on card 2's slice-typed `pack` (`pack != nil`), its per-card headings and its reworked `TestBeginBatch_KickstartPack`, and card 3 comes after card 2.
  Card 2's `renderLocate(stencilsDir, specsDir string, packOn bool)` changes no text, and card 3 edits only the locate fragment's stencil text, so the two agree.
- Fidelity: D1 and D2 are carried by card 2, D3 and D4 by card 3, D5 by card 1, and D6 by card 2.
  Each Constraint is met: new texts are registered stencils, the locate fragment keeps the spec pointer, refs come from `planparser` cards, the kind check sits in `planglyph`, and no card adds a top-level test.
  The pack-off, language-none and recovery byte identity is pinned by card 3's nil-pack and recovery rows, which use literal pre-change blocks, plus the begin-batch language-none row.
  No card adds unlicensed scope.
  `cardPointerPath` serves D2's same-display rule at two call sites.
- Test economy: card 1 adds a row and card 2 adds a subtest and rows to existing tests, each covering a behavior no existing assertion covers.
  These are the per-card repeat, the one-call resolve, the distinct log counts and the `none` entries.
  No new top-level test is planned.
- Attack surface and reachability: card 3's Bound paragraph states what the pack-on FRESH-READ text lets through and what bounds it.
  `kickstartPack`'s resolve error is reached by the missing-root subtest.
  `renderCardPack`'s errors are propagated stencil read and fill errors, with no refusal of their own.
- Verify coverage: `## verify:` builds, vets each tag, and runs the untagged and integration tiers over `internal/planindex`, `internal/planglyph`, `internal/websterengine` (which holds the `websterengine_test` package) and `contracts/stencils`, in the template's order.
