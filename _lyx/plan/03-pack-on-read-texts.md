# Card 3 — pack-on-read-texts

**Create:**
- `plan:internal/websterengine#renderReadRules` -> `func renderReadRules(stencilsDir string, packOn bool) (cardRead, freshRead string, err error)`
- `plan:contracts/stencils#WebsterFragmentCardReadPlain` -> `var WebsterFragmentCardReadPlain []byte`
- `plan:contracts/stencils#WebsterFragmentCardReadPack` -> `var WebsterFragmentCardReadPack []byte`
- `plan:contracts/stencils#WebsterFragmentFreshReadPlain` -> `var WebsterFragmentFreshReadPlain []byte`
- `plan:contracts/stencils#WebsterFragmentFreshReadPack` -> `var WebsterFragmentFreshReadPack []byte`
- `contracts/stencils/webster/webster-fragment-card-read-plain.md`
- `contracts/stencils/webster/webster-fragment-card-read-pack.md`
- `contracts/stencils/webster/webster-fragment-fresh-read-plain.md`
- `contracts/stencils/webster/webster-fragment-fresh-read-pack.md`

**Edit:**
- `contracts/stencils/webster/webster-body-implementer.md`
- `contracts/stencils/webster/webster-fragment-locate-pack.md`
- `contracts/stencils#entries`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils#wordingClaims`
- `.gitattributes`
- `internal/websterengine#RenderForkPrompt`
- `internal/websterengine#RenderRecoveryPrompt`
- `internal/websterengine/doc.go`
- `internal/websterengine_test#forkTemplateMarkerValues`
- `internal/websterengine_test#TestTemplates_FillRequiresEveryRequiredMarker`
- `internal/websterengine_test#TestRenderPrompts`
- `internal/websterengine_test#TestBeginBatch_KickstartPack`

**Intent:** Decisions D3 and D4: with a pack, each card starts with one parallel read of its card file and its listed spans, and the FRESH-READ rule names those spans; without a pack, every text stays byte-identical.

**Body markers.**
In `webster-body-implementer.md`, the FRESH-READ rule's body (the five lines from `Inherited context can be stale.` to `content you merely inherited is not.`) becomes the required marker `{{.fresh_read}}` under the unchanged `## The FRESH-READ rule` heading, and step 1's text (`Read the card file.` and its two continuation lines) becomes the required marker `{{.card_read}}`, so the line reads `1. {{.card_read}}`.

**Pack-off fragments** carry today's text verbatim, indentation included:
- `webster-fragment-fresh-read-plain`: the five FRESH-READ lines;
- `webster-fragment-card-read-plain`: today's three step-1 lines, from `Read the card file.` through the indented Intent-fallback line ending `matched by the same NN/slug.`

**Pack-on fragments**, continuation lines indented three spaces like step 1:
- `webster-fragment-card-read-pack` (D3): at the start of each card, read the card file and every span the kick-start pack lists under that card's heading in parallel, in one turn, before doing anything else, each span with the Read tool's `offset` at its start line and a `limit` reaching its end line; then: a span in a file an earlier card of this batch edited is not read by offset, that file is read whole in the same turn, and the member is found in it by the member name its glyph names, with the listed signature only a cross-check, since an earlier card may have re-signed it; then step 1's two sentences verbatim (`It is your whole instruction for that card.` and the Intent fallback to the Card Index line in `_lyx/plan/00-overview.md`), so the fragment ends on step 1's kept text.
- `webster-fragment-fresh-read-pack` (D4): inherited context can be stale, as today's first two sentences say; the one turn at each card's start (step 1) is that card's fresh read, and it re-reads the listed spans, not the whole files that hold them; in the turn right after it, before any edit, read whole every file the card's target list and `Uses:` list name that no listed span covers (a plain path, a self glyph's file); an unresolved member glyph's file is read whole in the turn right after the `lyx quarry resolve` call or grep that locates it, still before any edit; for a file an earlier card of this batch edited, the fresh read is step 1's whole-file read in that turn; only your own reads, taken now, are current.

**Locate fragment.**
`webster-fragment-locate-pack.md` keeps its rules and markers and changes these sentences:
- the opening says the kick-start pack below lists, under each card's path, where each member glyph on that card is, and step 1 read the card's listed spans (in place of "Open each listed file and read the listed span before editing.");
- the spans sentence becomes: spans are as of the batch's start, so in a file an earlier card of this batch edited, find the member by its glyph's member name, with the listed signature as a cross-check;
- a new sentence: the pack is a kick-start, not a complete list, so read further code, callers or whole files whenever the card needs it.
The no-`lyx quarry resolve`-or-grep rule for a resolved entry, the `plan:` sentence, the unresolved-member sentence and the `{{.specs_dir}}/loom/loom-plan-spec.md` pointer stay.

**Render.**
`renderReadRules` returns the trimmed text (`readFragment`) of the two pack-on fragments when `packOn` is true and the two pack-off fragments otherwise.
`RenderForkPrompt` fills `card_read` and `fresh_read` from `renderReadRules(stencilsDir, pack != nil)`, and `RenderRecoveryPrompt` from `renderReadRules(stencilsDir, false)`, so the recovery prompt and a fork without a pack (switch off, or a plan whose language opts out of glyphs) render the bytes they did before this task.
`stencils.go` embeds the four fragments with doc comments in the style of the locate fragments, registers them in `entries` after `webster-fragment-locate-pack`, and `.gitattributes` gets their `text eol=lf` rows; each fragment's banner says who reads it and which marker it fills.

**Bound (D4).**
Under the pack-on text a fork is no longer told to re-read whole the target and `Uses:` files that hold a listed span, so an edit outside a listed span in such a file is read only when the fork chooses to.
What bounds it: the member-name rule for a file an earlier card edited, the locate fragment's "read further whenever the card needs it", and the card gates and the batch gate, which still run every card's tests.
The pack-off text, which the recovery prompt always takes, keeps the whole-file re-read.

**Docs.**
`internal/websterengine/doc.go`'s `# the kick-start pack` section says step 1 and the FRESH-READ rule are markers filled per mode: without a pack the fragments carry the text as it stood before, with a pack step 1 reads the card file and its listed spans in one parallel turn (a file an earlier card edited whole), and FRESH-READ names the listed spans plus the uncovered target and `Uses:` files; it states the bound above.

**Tests.**
`forkTemplateMarkerValues` sets `card_read` and `fresh_read` placeholders, and `TestTemplates_FillRequiresEveryRequiredMarker` lists both as required for the fork and the recovery templates.

`TestRenderPrompts`:
- the nil-pack row "fork with a nil pack locates by quarry and renders the pre-pack bytes" also substitutes `{{.card_read}}` and `{{.fresh_read}}` with the pack-off fragments' trimmed text when building its pre-pack body, and its `contains` gains today's step-1 block and today's FRESH-READ body, written as string literals copied from today's `webster-body-implementer.md`, so the nil-pack prompt is pinned to the pre-change bytes, not just to the fragments;
- the existing row "recovery locates by quarry and carries no pack section" also builds the same substituted body, wants the recovery prompt equal to the one rendered from it, and wants the same two literal blocks;
- the multipart pack row also wants the pack-on step-1 phrase `in parallel, in one turn, before doing anything else`, the pack-on FRESH-READ phrase `not the whole files that hold them`, the uncovered-files phrase `that no listed span covers`, and the locate sentences on the member name and on reading beyond the pack; it wants not the pack-off FRESH-READ sentence starting `Before you edit anything, re-read`;
- the multipart row and the empty-pack row today assert the old locate opening `The pack below lists where each member glyph on the cards is.` (`template_test.go`, both rows' `contains`); both swap it for the reworded opening, and the nil-pack and recovery rows' `notContains` needle `The pack below lists` becomes a phrase of the reworded opening.

`TestBeginBatch_KickstartPack`: the switch-on row wants the pack-on step-1 phrase; the switch-off and language-none rows want `1. Read the card file.` and not the pack-on step-1 phrase.

`contracts/stencils`:
- `TestStencils_TopLevelMarkers` gets the four fragments with no marker;
- `wordingClaims`' `webster-body-implementer.md` entry claims `{{.card_read}}` and `{{.fresh_read}}` in place of the step-1 and fallback lines, which move to new entries pinning the pack-off fragments' lines verbatim;
- the `webster-fragment-card-read-pack.md` entry claims, each as its own `wantAll` string so dropping any one clause fails: `in parallel, in one turn, before doing anything else`; the Read tool's `` `offset` `` at the span's start line; a `` `limit` `` reaching its end line; the whole-file read in the same turn of a file an earlier card of this batch edited; finding the member by the member name its glyph names, with the listed signature only a cross-check; and step 1's two kept lines verbatim (`It is your whole instruction for that card.` and the Intent-fallback line);
- the `webster-fragment-fresh-read-pack.md` entry claims, each as its own string: `Inherited context can be stale.`; that the turn at each card's start is the card's fresh read; `not the whole files that hold them`; `that no listed span covers`, read whole in the turn right after, before any edit; an unresolved member glyph's file read whole in the turn right after the `lyx quarry resolve` call or grep that locates it; and, for a file an earlier card of this batch edited, the fresh read being step 1's whole-file read;
- the `webster-fragment-locate-pack.md` entry claims the reworded and new sentences, with a `wantNone` on the old `find the member by its listed signature.`

**ImpactSummary:** Touches only webster's implementer-body stencils and the two prompt renderers that fill them; the pack-off and recovery prompts keep their bytes, and the Master and verify-fix prompts are untouched.

**Commit:** 3: pack-on card read and FRESH-READ texts
