# Card 7 — Locate fragments and pack section in the fork prompt

**Create:**
- `contracts/stencils/webster/webster-fragment-locate-quarry.md`
- `contracts/stencils/webster/webster-fragment-locate-pack.md`
- `contracts/stencils/webster/webster-fragment-pack-section.md`
- `contracts/stencils/webster/webster-fragment-pack-empty.md`
- `contracts/stencils#WebsterFragmentLocateQuarry`
- `contracts/stencils#WebsterFragmentLocatePack`
- `contracts/stencils#WebsterFragmentPackSection`
- `contracts/stencils#WebsterFragmentPackEmpty`
- `internal/websterengine#renderPackSection`

**Edit:**
- `contracts/stencils/webster/webster-body-implementer.md`
- `contracts/stencils#entries`
- `contracts/stencils#wordingClaims`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `internal/websterengine#RenderForkPrompt` -> `func RenderForkPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, reportPath, planDir, promptWorktreeRoot, stencilsDir, specsDir, outcomePath, summaryPath string, selfFixCap int, notePath string, pack *planindex.Pack) ([]byte, error)`
- `internal/websterengine#RenderRecoveryPrompt`
- `internal/websterengine#BeginBatch`
- `internal/websterengine_test#renderFork` -> `func renderFork(batch batcher.Batch, root, stencilsDir, specsDir, prevDigest string, selfFixCap int, notePath string, pack *planindex.Pack) (string, error)`
- `internal/websterengine_test#forkTemplateMarkerValues`
- `internal/websterengine_test#recoveryTemplateMarkerValues`
- `internal/websterengine_test#TestTemplates_FillRequiresEveryRequiredMarker`
- `internal/websterengine_test#TestRenderPrompts`
- `internal/websterengine_test#TestRenderPrompts_ParentDirective`
- `internal/websterengine_test#TestRenderPrompts_FrictionDirective`

**Uses:**
- `internal/planindex#Pack`

**Intent:**
Decisions D7 and D8: the fork prompt's locate step and its pack section become stencils.
`RenderForkPrompt` picks one by its pack argument.
Card 8 passes a real pack.

Stencils (`PATTERN-stencil-ownership`):
- Move the two locate sentences of step 2 in `webster-body-implementer.md` verbatim into `webster-fragment-locate-quarry.md`, keeping their indentation.
  These are "Locate them with one `lyx quarry resolve <glyph>...` call…" and "A `plan:` handle names a member the card creates and does not resolve yet."
  In the body they are replaced by a required `{{.locate}}` marker.
- `webster-fragment-locate-pack.md` holds the pack fragment D7 words:
  - The pack below lists where each member glyph on the cards is.
  - Open each listed file and read the listed span before editing.
  - Do not call `lyx quarry resolve`, and do not grep, for a resolved entry.
  - Spans are as of the batch's start, so in a file already edited in this batch, find the member by its listed signature.
  - A `plan:` entry names a member a card creates and does not resolve yet, so there is nothing to locate.
  - An unresolved member glyph is located with `lyx quarry resolve` or a grep, since a member glyph names a package, not a file.
  - Card grammar is in `{{.specs_dir}}/loom/loom-plan-spec.md`.
  The fragment points there for card grammar and never restates it (`PATTERN-producer-pointer-rule`).
- `webster-fragment-pack-section.md` holds the pack's fixed heading and lead-in, the `{{.pack_entries}}` marker, an unresolved heading with its lead-in, and the `{{.pack_unresolved}}` marker.
- `webster-fragment-pack-empty.md` holds D8's one line, "no member glyphs on this batch's cards".
- The body gains an optional `{{.kickstart_pack}}` marker where the section renders.
  Place it so that an empty fill leaves no byte behind.
- Each new stencil carries a banner comment like its siblings and is registered as an embedded default in `contracts/stencils/stencils.go`: a `//go:embed` var and an `entries` row beside the other webster stencils.
- The quarry fragment and the empty-pack fragment are marker-free, since they are filled verbatim into a marker.
  The pack fragment's one marker is `specs_dir`.

Rendering in `internal/websterengine/render.go`:
- `RenderForkPrompt` takes a trailing `pack *planindex.Pack`.
- With a nil pack (switch off, or `language: none`), it fills `locate` with the quarry fragment's banner-stripped text and leaves `kickstart_pack` empty.
- With a non-nil pack, it fills the pack fragment's `specs_dir` with `specsDir` through `stencil.Fill`, fills `locate` with the result, and fills `kickstart_pack` with `renderPackSection`'s output.
- `RenderRecoveryPrompt` always fills `locate` with the quarry fragment and never fills `kickstart_pack`.
  Its signature is unchanged.
- Both prompts list `kickstart_pack` as optional to `stencil.FillOptional`.

`renderPackSection` reads `webster-fragment-pack-section` through `stencilstore` and fills its two markers with Go-rendered lists only:
- `pack_entries` is one bullet per resolved entry with its glyph.
  Under each bullet, one sub-bullet per declaration gives `file:start-end` and the signature.
- When the pack holds no member glyph at all, resolved or unresolved, `pack_entries` is the banner-stripped text of `webster-fragment-pack-empty`, read through `stencilstore` (D8).
  `planparser.IsHandleRef` tells a `plan:` ref from a member glyph.
- `pack_unresolved` is one bullet per unresolved ref with its reason, or `none`.

`BeginBatch` passes a nil pack on this card.
The fork prompt keeps rendering exactly as before until card 8 builds a pack.

Tests in `internal/websterengine/template_test.go`:
- `renderFork` takes the pack.
- `TestRenderPrompts_FrictionDirective` passes a nil pack.
- `forkTemplateMarkerValues` and `recoveryTemplateMarkerValues` supply `locate` and `kickstart_pack`.
- `TestTemplates_FillRequiresEveryRequiredMarker` lists `locate` as required and `kickstart_pack` as optional for fork and recovery.
- `TestRenderPrompts_ParentDirective` passes a nil pack.
- `TestRenderPrompts` gains these rows:
  - Nil pack: the fork render contains the quarry fragment's sentences and no pack heading.
    It equals byte for byte the render from a stencils directory whose body has the `{{.kickstart_pack}}` marker removed and the `{{.locate}}` marker replaced by the quarry fragment's text.
    Together with the verbatim claim below, that pins the off render to today's bytes.
  - A pack with a resolved multipart entry: the pack fragment and the section list each glyph with file, span and signature, in pack order.
    The quarry fragment's "Locate them with one" sentence is absent.
  - A pack with an unresolved member glyph and a `plan:` handle: both are under the unresolved heading with their reasons.
  - An empty pack: the section heading and the D8 line render, and the pack fragment is used.
  - A recovery render contains the quarry fragment and no pack heading.

Tests in `contracts/stencils`:
- In `wordingClaims`, the `webster-body-implementer.md` row's quarry-locate claim moves to a new `webster-fragment-locate-quarry.md` row, pinning both sentences in full.
  The body row instead pins `{{.locate}}` and `{{.kickstart_pack}}`.
- A new `webster-fragment-locate-pack.md` row pins each D7 instruction.
  This includes that a `plan:` entry has nothing to locate, that an unresolved member glyph is located with `lyx quarry resolve` or a grep, and the `loom-plan-spec.md` pointer.
- A new `webster-fragment-pack-empty.md` row pins the D8 line.
- `TestStencils_TopLevelMarkers` gains rows for the four new stencils: none for the quarry and empty-pack fragments, `specs_dir` for the pack fragment, and `pack_entries` and `pack_unresolved` for the section.
- `TestRegistry_MatchesOnDiskTree` already pins the registration.

**ImpactSummary:** `RenderForkPrompt` gains a parameter whose callers (`BeginBatch` and the template tests) change on this card; with a nil pack the fork prompt and the recovery prompt render the same bytes as before, and four stencils join the registry.
