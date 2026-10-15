# Card 3 — per-card-layout

**Create:**
- `contracts/stencils/webster/webster-fragment-fresh-read-pack-card.md`
- `contracts/stencils/webster/webster-fragment-card-read-pack-card.md`
- `contracts/stencils/webster/webster-fragment-locate-pack-card.md`
- `contracts/stencils/webster/webster-fragment-pack-start-card.md`
- `contracts/stencils/webster/webster-fragment-card-pack-allowed.md`
- `plan:contracts/stencils#WebsterFragmentFreshReadPackCard` -> `var WebsterFragmentFreshReadPackCard []byte`
- `plan:contracts/stencils#WebsterFragmentCardReadPackCard` -> `var WebsterFragmentCardReadPackCard []byte`
- `plan:contracts/stencils#WebsterFragmentLocatePackCard` -> `var WebsterFragmentLocatePackCard []byte`
- `plan:contracts/stencils#WebsterFragmentPackStartCard` -> `var WebsterFragmentPackStartCard []byte`
- `plan:contracts/stencils#WebsterFragmentCardPackAllowed` -> `var WebsterFragmentCardPackAllowed []byte`
- `plan:internal/websterengine#PackLayout` -> `type PackLayout string`
- `plan:internal/websterengine#PackLayoutFile` -> `const PackLayoutFile PackLayout = "file"`
- `plan:internal/websterengine#PackLayoutInline` -> `const PackLayoutInline PackLayout = "inline"`
- `plan:internal/websterengine#PackLayoutCard` -> `const PackLayoutCard PackLayout = "card"`
- `plan:internal/websterengine#packLayoutOf` -> `func packLayoutOf(cfg Config, geom Geometry) PackLayout`

**Edit:**
- `contracts/stencils/webster/webster-body-implementer.md`
- `contracts/stencils/webster/webster-fragment-card-read-pack.md`
- `contracts/stencils/webster/webster-fragment-pack-read.md`
- `contracts/stencils/webster/webster-fragment-locate-pack.md`
- `contracts/stencils/webster/webster-fragment-fresh-read-pack-inline.md`
- `contracts/stencils/webster/webster-fragment-locate-pack-inline.md`
- `contracts/stencils/webster/webster-fragment-pack-section-inline.md`
- `contracts/stencils#WebsterFragmentCardReadPack`
- `contracts/stencils#WebsterFragmentPackRead`
- `contracts/stencils#entries`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils#wordingClaims`
- `.gitattributes`
- `internal/websterengine#renderLocate` -> `func renderLocate(stencilsDir, specsDir string, packOn bool, layout PackLayout) (string, error)`
- `internal/websterengine#renderReadRules` -> `func renderReadRules(stencilsDir string, packOn bool, layout PackLayout) (cardRead, freshRead string, err error)`
- `internal/websterengine#RenderForkPrompt` -> `func RenderForkPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, reportPath, planDir, promptWorktreeRoot, stencilsDir, specsDir, outcomePath, summaryPath string, selfFixCap int, notePath, packPath string, layout PackLayout, pack []planindex.CardPack) (prompt, packFile []byte, err error)`
- `internal/websterengine#RenderRecoveryPrompt`
- `internal/websterengine#buildKickstartPack`
- `internal/websterengine#BeginBatch`
- `internal/websterengine/doc.go`
- `internal/websterengine_test#forkTemplateMarkerValues`
- `internal/websterengine_test#TestTemplates_FillRequiresEveryRequiredMarker`
- `internal/websterengine_test#renderFork`
- `internal/websterengine_test#renderForkWithPack` -> `func renderForkWithPack(batch batcher.Batch, root, stencilsDir, specsDir, prevDigest string, selfFixCap int, notePath string, layout websterengine.PackLayout, pack []planindex.CardPack) (prompt, packFile string, err error)`
- `internal/websterengine_test#renderForkPromptAndPack`
- `internal/websterengine_test#writePrePackBody`
- `internal/websterengine_test#TestRenderPrompts`
- `internal/websterengine_test#TestBeginBatch_KickstartPack`

**Uses:**
- `internal/websterengine#Config`
- `internal/websterengine#Geometry`

**Intent:**
Render the per-card layout in the implementer fork prompt and make begin-batch pick it when `kickstart_pack_per_card` is true (D1, D2, D4, D5).
The file layout, the inline layout, the pack-off prompt and the recovery prompt render the bytes they do today.

Five new fragments, each with a banner in the style of its neighbours (what reads it, which marker it fills, its markers), each registered in `stencils.go` as an embedded var with a doc comment, in `entries` after the inline-layout rows, and in `.gitattributes` with `text eol=lf`:
- `webster-fragment-fresh-read-pack-card`, filling `fresh_read`, no marker: the first four sentences of the file-layout FRESH-READ rule as they stand (inherited context can be stale, through "content you merely inherited is not."), then "Your first work turn reads in parallel every card file the card list below points at, and nothing else: they are the batch's whole plan, and they hold no code.", then "What you read before each card's first edit is set out in that card's step 1.", then the Edit-only sentence the file-layout rule ends with.
- `webster-fragment-card-read-pack-card`, filling `card_read`, no marker: "Before this card's first edit, run `lyx loom card-pack <N>` with this card's number, then read in one parallel turn every span it lists, each with the Read tool's `offset` at its start line and a `limit` reaching its end line." then, as continuation lines indented three spaces like the pack-on fragment's, "Then read whole every existing file the card's target list or `Uses:` list names that no listed span covers, and locate every unresolved member glyph that is not a `plan:` handle, then read its file whole." and "The card file, which your first turn read, is your whole instruction for that card." and the Card Index fallback sentence the pack-on fragment carries.
- `webster-fragment-locate-pack-card`, filling `locate`, marker `specs_dir`, each line indented three spaces: the card pack, taken at this card's start, lists where each member glyph on the card is, and those spans are complete for a start, so read further code, callers or whole files only when the card needs it; do not call `lyx quarry resolve`, and do not grep, for a listed entry; the card pack's line numbers hold until this card's first edit only, after which code is located by its text and never read by those lines; a `plan:` entry names a member a card creates, so there is nothing to locate; an unresolved member glyph is located with `lyx quarry resolve` or a grep, since a member glyph names a package, not a file; when the `lyx loom card-pack` call fails, every glyph on the card is located that same way; card grammar is in `{{.specs_dir}}/loom/loom-plan-spec.md`.
- `webster-fragment-pack-start-card`, filling `kickstart_pack`, no marker: `## Start here`, a blank line, then "Now read every card file the card list points at, in one parallel turn, and then begin the first card at its step 1."
- `webster-fragment-card-pack-allowed`, filling the new `card_pack_allowed` marker, no marker: the text ``, `lyx loom card-pack` `` (a comma, a space and the backticked command), with no trailing newline once read.

The banners of `webster-fragment-card-read-pack` and `webster-fragment-pack-read`, and the doc comments of `WebsterFragmentCardReadPack` and `WebsterFragmentPackRead`, say they are read for the file and inline layouts, and for the file layout alone, since a per-card pack fills `card_read` and `kickstart_pack` from the new fragments.
The banners of `webster-fragment-locate-pack` and the three inline fragments `webster-fragment-fresh-read-pack-inline`, `webster-fragment-locate-pack-inline` and `webster-fragment-pack-section-inline` read "kickstart_pack_per_card is false and kickstart_pack_file is true" for the file layout and "kickstart_pack_per_card is false and kickstart_pack_file is false" for the inline layout, and the text after each banner stays byte-identical.

`webster-body-implementer.md`'s sandbox paragraph becomes "`lyx loom lint-pattern`{{.card_pack_allowed}} and `lyx quarry resolve` are always allowed", the marker directly after `` `lyx loom lint-pattern` `` and before ` and`, so a per-card prompt lists lint-comments, lint-pattern, card-pack and quarry resolve, and every other prompt fills it empty.

`PackLayout` (`render.go`) names the three layouts, its string values the `pack_layout` trace values; `packLayoutOf` returns `PackLayoutCard` when `cfg.KickstartPackPerCard` and `geom.AnchorRoot` equals `geom.WorktreeRoot`, else `PackLayoutFile` when `cfg.KickstartPackFile`, else `PackLayoutInline`.
A standalone geometry sets `AnchorRoot` to the state directory and `WorktreeRoot` to the target, where `lyx loom card-pack`, a hub-only verb, cannot run, so a standalone run keeps the file or inline layout it renders today and never gets a prompt whose every card-pack call refuses; a hub geometry sets both to the anchor path.
`renderLocate` and `renderReadRules` take the layout in place of the inline bool: with packOn, `PackLayoutInline` reads the frozen `-inline` fragments as today, `PackLayoutCard` reads the `-pack-card` fragments (locate filled with specs_dir like the other pack locates; card_read from `webster-fragment-card-read-pack-card`), and `PackLayoutFile` reads today's fragments; without packOn the layout is ignored.
`RenderForkPrompt` takes `layout PackLayout` in place of `packFileLayout bool`.
With a non-nil pack and `PackLayoutCard` it fills `kickstart_pack` with "\n\n" and the `webster-fragment-pack-start-card` text, `card_pack_allowed` with the `webster-fragment-card-pack-allowed` text, leaves `kickstart_pack_inline` empty and returns a nil packFile, so begin-batch writes no `<batch>-pack.md`; the pack's entries reach neither the prompt nor any file.
The file and inline branches are unchanged; every branch but the per-card one fills `card_pack_allowed` empty, and a nil pack renders the pack-off prompt whatever the layout.
`card_pack_allowed` joins the optional markers of `RenderForkPrompt`'s and `RenderRecoveryPrompt`'s `FillOptional` calls, and the recovery prompt fills it empty, so its bytes are unchanged.
`buildKickstartPack` logs `pack_layout` as `string(packLayoutOf(deps.Config, deps.Geom))` when a pack is built and `none` otherwise, so a per-card batch's line ends `pack_layout=card`; it still builds the batch pack under the per-card layout, so the off, language-none and empty outcomes and the resolved and unresolved counts keep their batch-start meaning.
`BeginBatch` passes `packLayoutOf(deps.Config, deps.Geom)`; a pack file an earlier file-layout call of the same batch wrote stays on disk and the per-card prompt never names it.
`internal/websterengine/doc.go`'s kick-start pack section describes the per-card layout, the key's precedence over `kickstart_pack_file` in a hub geometry and its fall-back to that key in a standalone one, the card files as the batch overview, the `## Start here` section, the sandbox marker, and `card` among the `pack_layout` values.

Tests:
- `TestStencils_TopLevelMarkers` gains a row per new fragment: `specs_dir` for the locate fragment, none for the other four.
- `wordingClaims` gains a row per new fragment pinning its load-bearing sentences: the first work turn reads every card file and nothing else; step 1 names `lyx loom card-pack <N>` before the first edit and the one-turn span read with `offset` and `limit`; the locate step's line numbers holding until the card's first edit only and its fallback for a failed `card-pack` call; the closing section's `## Start here` heading and read-every-card-file line; the allowed fragment's exact text.
  The body row's sandbox claim becomes "`lyx loom lint-pattern`{{.card_pack_allowed}} and `lyx quarry resolve` are always allowed from any strand", since the old needle no longer occurs.
- `forkTemplateMarkerValues` gains `card_pack_allowed`, and `TestTemplates_FillRequiresEveryRequiredMarker` lists it among the fork's and the recovery's optional markers.
- `writePrePackBody` also replaces `{{.card_pack_allowed}}` with nothing, so the pack-off and recovery rows keep comparing against the pre-pack body.
- `renderForkWithPack` takes the layout; `renderFork` and `renderForkPromptAndPack` pass `websterengine.PackLayoutFile`, and `TestRenderPrompts`' rows pass `PackLayoutFile` or `PackLayoutInline` where they passed true or false; `TestRenderPrompts_ParentDirective` calls only `renderFork`, whose signature stays, so it does not change.
- `TestRenderPrompts`: the nil-pack row also renders under `PackLayoutCard` and requires the same bytes as under the file layout and no pack file;
  the multipart row's file-layout prompt and pack file stay pinned to `multipartPackFile` and to the render of a body without the inline marker, and the row also asserts the file-layout prompt holds no `lyx loom card-pack`, so the file layout's bytes stay pinned;
  the inline row stays pinned to the `1e39afc10^` constants and also asserts no `lyx loom card-pack` text;
  the recovery row renders the same bytes as before and holds no `lyx loom card-pack`;
  a new row "fork with a per-card pack reads the card files first and fetches each card's pack" renders `PackLayoutCard` over a one-entry pack and asserts a nil packFile, the per-card FRESH-READ, step 1 and locate texts in place, the sandbox sentence reading "`lyx loom lint-comments`, `lyx loom lint-pattern`, `lyx loom card-pack` and `lyx quarry resolve` are always allowed", the prompt ending with the `## Start here` section, and no `## Kick-start pack`, no pack entry line and no pack path;
  a new row "fork with a per-card pack refuses a missing per-card start stencil" removes `webster-fragment-pack-start-card` from the stencils directory and wants an error, as the inline row does for its section stencil.
- `TestBeginBatch_KickstartPack` gains a `perCard bool` case field that sets `Config.KickstartPackPerCard` before the first call, and rows:
  "the per-card layout writes no pack file and names card-pack" (perCard, noPack, wantPrompt the per-card FRESH-READ and step 1 needles and `## Start here`, wantNotPrompt `packFirstTurn`, the pack entry lines and `packHeading`, log `batch=1 kickstart_pack=on resolved=1 unresolved=0 pack_layout=card`);
  "the per-card switch overrides kickstart_pack_file" (perCard with `inline` set, noPack, the same needles and the same full log line);
  "a flip from file to per-card leaves a prompt that names no pack file" (flip toggles `KickstartPackPerCard` when perCard-flip is set, stalePack, log `batch=1 kickstart_pack=on resolved=1 unresolved=0 pack_layout=file` then the same line ending `pack_layout=card`);
  "a flip from per-card to file writes the pack file" (the reverse, the `pack_layout=card` line then the `pack_layout=file` one);
  "a standalone geometry keeps the file layout under per-card" (perCard and a new `standaloneGeom bool` case field that sets `Geom.AnchorRoot` to a fresh temp directory apart from `WorktreeRoot`, wantPrompt `packFirstTurn` and the pack entry lines, wantNotPrompt the per-card step 1 needle, log `batch=1 kickstart_pack=on resolved=1 unresolved=0 pack_layout=file`), which writes the pack file.
  The case gains a `flipPerCard bool` field naming which key `flip` toggles, so the existing flip rows keep flipping `kickstart_pack_file`.
  `layoutBlind`'s second call flips both `KickstartPackFile` and `KickstartPackPerCard`, so it compares the file layout's settings against the per-card ones, and the off, language-none and empty rows all set `layoutBlind` (today only the off row does), each requiring the same prompt bytes, no pack file and a `pack_layout=none` line on both calls: the pack-off prompt holds under every layout value at begin-batch, and the nil-pack `TestRenderPrompts` row covers the inline value.

**ImpactSummary:** RenderForkPrompt, renderLocate and renderReadRules take a PackLayout in place of a bool, with BeginBatch and the template_test helpers their only callers; the file, inline, pack-off and recovery bytes are pinned unchanged and only a per-card config in a hub geometry changes a prompt.

**Commit:** 3: webster: per-card kick-start pack layout in the fork prompt
