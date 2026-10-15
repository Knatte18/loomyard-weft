# Card 3 — render-layout

**Edit:**
- `contracts/stencils/webster/webster-body-implementer.md`
- `contracts/stencils#wordingClaims`
- `internal/websterengine#renderLocate` -> `func renderLocate(stencilsDir, specsDir string, packOn, inline bool) (string, error)`
- `internal/websterengine#renderReadRules` -> `func renderReadRules(stencilsDir string, packOn, inline bool) (cardRead, freshRead string, err error)`
- `internal/websterengine#renderPackSection` -> `func renderPackSection(stencilsDir, sectionStencil string, pack []planindex.CardPack, planDirDisplay string) (string, error)`
- `internal/websterengine#RenderForkPrompt` -> `func RenderForkPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, reportPath, planDir, promptWorktreeRoot, stencilsDir, specsDir, outcomePath, summaryPath string, selfFixCap int, notePath, packPath string, packFileLayout bool, pack []planindex.CardPack) (prompt, packFile []byte, err error)`
- `internal/websterengine#RenderRecoveryPrompt`
- `internal/websterengine#BeginBatch`
- `internal/websterengine/doc.go`
- `internal/websterengine_test#forkTemplateMarkerValues`
- `internal/websterengine_test#TestTemplates_FillRequiresEveryRequiredMarker`
- `internal/websterengine_test#writePrePackBody`
- `internal/websterengine_test#renderFork`
- `internal/websterengine_test#renderForkWithPack` -> `func renderForkWithPack(batch batcher.Batch, root, stencilsDir, specsDir, prevDigest string, selfFixCap int, notePath string, packFileLayout bool, pack []planindex.CardPack) (prompt, packFile string, err error)`
- `internal/websterengine_test#renderForkPromptAndPack`
- `internal/websterengine_test#TestRenderPrompts`
- `internal/websterengine_test#TestBeginBatch_KickstartPack`

**Intent:** Render the layout `kickstart_pack_file` picks (D3, D4).

Body (D3): `webster-body-implementer.md` gains the optional marker `{{.kickstart_pack_inline}}` at the end of the per-card closing paragraph, right after "… do not continue to a later card on top of a broken one.", before the blank line and `## The sandbox rule`, where `1e39afc10^` had `{{.kickstart_pack}}`.
The end-of-body `{{.kickstart_pack}}` marker stays.
`wordingClaims`' `webster-body-implementer.md` row gains a `wantAll` claim, why "the inline pack section sits after the per-card closing paragraph and before the sandbox rule, as at 1e39afc10^", needle "do not continue to a later card on top of a broken one.{{.kickstart_pack_inline}}\n\n## The sandbox rule"; the existing end-of-body `{{.kickstart_pack}}` claim stays.

Render (D4):

- `renderLocate` with `packOn` and `inline` reads `webster-fragment-locate-pack-inline`, else as today; `inline` is ignored when `packOn` is false.
- `renderReadRules` with `packOn` and `inline` fills `fresh_read` from `webster-fragment-fresh-read-pack-inline`; `card_read` stays `webster-fragment-card-read-pack` in both layouts (D2), and `packOn` false reads the plain fragments as today.
- `renderPackSection` reads the stencil named `sectionStencil` in place of the literal `webster-fragment-pack-section`.
- The doc comments of `renderLocate`, `renderReadRules` and `renderPackSection` name the inline variant beside today's choices; `renderPackSection`'s says it renders the pack file's content in the file layout and the prompt's inline section, which has no read instruction, in the inline layout.
- `RenderForkPrompt` takes `packFileLayout`, from `deps.Config.KickstartPackFile` at `BeginBatch`'s call site.
  With a nil pack it renders as today under either value.
  With a pack and `packFileLayout` true it renders as today: `kickstart_pack` holds the read line, `kickstart_pack_inline` fills empty, and it returns the pack file.
  With a pack and `packFileLayout` false it passes `inline` true to `renderLocate` and `renderReadRules`, renders `webster-fragment-pack-section-inline` and fills `"\n\n" + section` into `kickstart_pack_inline` (the marker ends a line, so the section brings its blank-line separation and an empty fill leaves no byte), fills `kickstart_pack` empty, never calls `renderPackRead`, and returns a nil `packFile`.
  `kickstart_pack_inline` joins the `FillOptional` optional list.
  Its doc comment states both layouts.
- `RenderRecoveryPrompt` passes `inline` false to both helpers and adds `"kickstart_pack_inline"` to its `FillOptional` optional list with no value, so the recovery prompt renders the bytes it does today.
- `BeginBatch` passes `deps.Config.KickstartPackFile`; its existing nil-`packFile` branch then writes no `<batch>-pack.md` for the inline layout.
  A pack file an earlier file-layout call of the same batch wrote stays on disk, and the inline prompt never names it.
  The empty-pack rule is decided in `buildKickstartPack` before the layout matters, so it holds in both layouts.
  Its doc comment's "preceded by the kick-start pack file's beside it when the batch has a pack" becomes a clause saying the pack file is written beside the prompt only when the batch has a pack and `kickstart_pack_file` is true, the file layout.

Docs: `doc.go`'s "the kick-start pack" section describes `kickstart_pack_file`, its default, both layouts (the inline one being the `1e39afc10^` text kept in three frozen fragments, its pack section between the per-card steps and the sandbox rule, no pack file and no `## Start here`), that a flip applies from the next begin-batch on, and that a stale pack file from an earlier file-layout call is left on disk unnamed.

Tests (`template_test.go`):

- `forkTemplateMarkerValues` gains `"kickstart_pack_inline": "\n\n## Kick-start pack\n\nthe inline pack."`; `TestTemplates_FillRequiresEveryRequiredMarker` adds `"kickstart_pack_inline"` to the fork and recovery `optional` lists.
- `writePrePackBody` adds `"{{.kickstart_pack_inline}}", ""` to its replacer, so the pre-pack rows keep comparing against today's bytes.
- `renderForkWithPack` takes `packFileLayout` and passes it through; `renderFork` and `renderForkPromptAndPack` pass `true`.
- `TestRenderPrompts`:
  the "fork with a nil pack" row also renders with `packFileLayout` false and requires the same prompt bytes and an empty pack file (acceptance: pack off renders today's bytes under either value);
  the "fork with a resolved multipart pack" row also renders the same batch and pack against a second `testLayout` stencils dir whose body has `{{.kickstart_pack_inline}}` replaced by "" (the body as it stands before this card) and requires prompt and pack file byte-equal to the first render, and it requires the pack file equal to a literal of today's pack file for that pack, copied from a render at the commit before this card, so a change to `renderPackSection` that both renders share still fails (acceptance: the file layout is byte-identical to today's);
  a new row "fork with an inline pack carries the 1e39afc10^ texts and the pack before the sandbox rule" renders the multipart row's batch and pack with `renderForkWithPack(…, false, pack)` and requires an empty pack file, `notContains` "## Start here", `testPackPath`, "kick-start pack file", "Read all of this in one turn" and "{{", and `contains` three exact strings built from the `1e39afc10^` bodies (banner and trailing newline stripped, as `git show 1e39afc10^:contracts/stencils/webster/<fragment>.md` gives them):
  "## The FRESH-READ rule\n\n" + the fresh-read-pack body + "\n\n## Prior-batch context";
  "in exactly the targets its type labels name.\n" + the locate-pack body with `{{.specs_dir}}` replaced by the row's specs dir + "\n3. Run each step";
  "on top of a broken one.\n\n" + the pack-section body up to and excluding `{{.pack_cards}}` + "### `_lyx/plan/01-alpha.md`";
  and "1. The card file, which your first turn read, is your whole instruction for that card.\n" (D2: step 1 is the same in both layouts), with "1. Read the card file." in `notContains`;
  and `inOrder` "on top of a broken one.", "## Kick-start pack", "### `_lyx/plan/01-alpha.md`", "  - `internal/foo/c.go:1-2` — func Baz()", "## The sandbox rule", "## Your final action: the minimal batch-report".
  A new row "fork with an inline pack refuses a missing inline section stencil" removes `webster-fragment-pack-section-inline` from a `testLayout` stencils dir, renders the inline layout with a pack, and expects `wantErr`; the locate and FRESH-READ inline reads propagate a `stencilstore.Read` error through the same wraps as their file-layout siblings.
  The recovery row is unchanged and keeps pinning today's recovery bytes through `writePrePackBody`.

Tests (`beginbatch_test.go`, `TestBeginBatch_KickstartPack`):

- New case fields: `inline bool` sets `fx.Deps.Config.KickstartPackFile = !tt.inline` before the first call; `flip bool` makes the case call `BeginBatch(fx.Deps, 1)` once, flip `KickstartPackFile`, call it again and assert on the second call's files; `stalePack bool` expects the pack file present but the prompt naming neither the read line nor the pack path, and checks the `wantPrompt` and `wantNotPrompt` needles against the prompt alone, since the stale pack file holds `## Kick-start pack` too.
  `noPack`'s comment becomes "begin-batch writes no pack file".
- New rows:
  "the inline layout carries the pack in the prompt and writes no pack file" (`inline`, `noPack`, `wantPrompt` `packHeading`, the `resolvedBaz` lines and "Your first turn, before any edit, reads in parallel", `wantNotPrompt` "## Start here", `packFirstTurn`);
  "a flip from inline to file writes the pack file" (`inline`, `flip`; the existing with-pack branch asserts the pack file and the read line on the second call; that the pack file is written before the prompt rests on `BeginBatch`'s unchanged `packFile != nil` write ahead of the prompt write, which no test can observe after the call returns);
  "a flip from file to inline leaves a prompt that names no pack file" (`flip`, `stalePack`, `wantPrompt` `packHeading` and "Your first turn, before any edit, reads in parallel", `wantNotPrompt` "## Start here").
- A further field `layoutBlind bool`, set on the existing switch-off row, makes the case also call `BeginBatch` again with `KickstartPackFile` flipped and requires the second prompt byte-equal to the first and still no pack file (acceptance: a prompt without a pack renders the same bytes under either value).
  The language-none and empty-pack rows reach `RenderForkPrompt` with the same nil pack (`buildKickstartPack` returns nil at each of its three exits before the layout is read), and the render row "fork with a nil pack" pins that nil-pack branch under both values, so they take no `layoutBlind`.

"Every begin-batch reads it afresh" (D1) has two pinned halves: `TestLoadConfig` pins that the loader reads the key from the layer stack, and the flip rows pin that `BeginBatch` renders from the `Config` it is handed on each call.
Each `lyx webster begin-batch` runs as its own process that loads `webster.yaml` at `internal/webstercli/wiring.go`'s `websterengine.LoadConfig` call, which this plan leaves unchanged, as the existing `kickstart_pack` key relies on.

**ImpactSummary:** `RenderForkPrompt` gains one parameter with one production caller (`BeginBatch`) and one test helper; the three render helpers are package-private with callers only in `render.go`; the recovery prompt and every pack-off and file-layout prompt keep today's bytes.

**Commit:** 3: webster renders the pack inline when kickstart_pack_file is false
