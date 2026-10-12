# Card 2 — pack-per-card

**Create:**
- `plan:internal/planindex#CardPack` -> `type CardPack struct`
- `plan:internal/planglyph#kickstartPack` -> `func kickstartPack(plan *planparser.Plan, cards []planparser.Card, resolve func(targets []string) ([]quarry.ResolveResult, error)) ([]planindex.CardPack, error)`
- `plan:internal/websterengine#cardPointerPath` -> `func cardPointerPath(c planparser.Card, planDirDisplay string) string`
- `plan:internal/websterengine#renderCardPack` -> `func renderCardPack(stencilsDir, cardPath string, pack planindex.Pack) (string, error)`
- `plan:contracts/stencils#WebsterFragmentPackCard` -> `var WebsterFragmentPackCard []byte`
- `plan:contracts/stencils#WebsterFragmentPackUnresolved` -> `var WebsterFragmentPackUnresolved []byte`
- `contracts/stencils/webster/webster-fragment-pack-card.md`
- `contracts/stencils/webster/webster-fragment-pack-unresolved.md`

**Edit:**
- `internal/planindex#Pack`
- `internal/planindex#Index.KickstartPack`
- `internal/planglyph#KickstartPack` -> `func KickstartPack(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string) ([]planindex.CardPack, error)`
- `internal/planglyph#index.KickstartPack` -> `func (index) KickstartPack(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string) ([]planindex.CardPack, error)`
- `internal/planglyph#selectPackRefs` -> `func selectPackRefs(card planparser.Card) []packRef`
- `internal/planglyph#TestKickstartPack`
- `internal/planglyph/doc.go`
- `internal/websterengine#buildKickstartPack` -> `func buildKickstartPack(deps BeginDeps, index planindex.Index, batch batcher.Batch, number int) []planindex.CardPack`
- `internal/websterengine#BeginBatch`
- `internal/websterengine#RenderForkPrompt` -> `func RenderForkPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, reportPath, planDir, promptWorktreeRoot, stencilsDir, specsDir, outcomePath, summaryPath string, selfFixCap int, notePath string, pack []planindex.CardPack) ([]byte, error)`
- `internal/websterengine#RenderRecoveryPrompt`
- `internal/websterengine#renderLocate` -> `func renderLocate(stencilsDir, specsDir string, packOn bool) (string, error)`
- `internal/websterengine#renderPackSection` -> `func renderPackSection(stencilsDir string, pack []planindex.CardPack, planDirDisplay string) (string, error)`
- `internal/websterengine#renderCardPointers`
- `internal/websterengine#TestPlanDirDisplay`
- `internal/websterengine/doc.go`
- `internal/websterengine_test#fakeIndex`
- `internal/websterengine_test#fakeIndex.KickstartPack` -> `func (i *fakeIndex) KickstartPack(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string) ([]planindex.CardPack, error)`
- `internal/websterengine_test#TestBeginBatch_KickstartPack`
- `internal/websterengine_test#renderFork` -> `func renderFork(batch batcher.Batch, root, stencilsDir, specsDir, prevDigest string, selfFixCap int, notePath string, pack []planindex.CardPack) (string, error)`
- `internal/websterengine_test#TestRenderPrompts`
- `internal/websterengine_test#TestRenderPrompts_FrictionDirective`
- `internal/websterengine_test#TestRenderPrompts_ParentDirective`
- `contracts/stencils/webster/webster-fragment-pack-section.md`
- `contracts/stencils/webster/webster-fragment-pack-empty.md`
- `contracts/stencils#WebsterFragmentPackSection`
- `contracts/stencils#WebsterFragmentPackEmpty`
- `contracts/stencils#entries`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils#wordingClaims`
- `.gitattributes`

**Intent:** Decisions D1, D2 and D6: the kick-start pack is grouped per card, and the fork prompt renders one subsection per card.

**Shape (D1).**
`planindex` declares `CardPack` with two fields, `Card planparser.Card` and `Pack Pack`: one card paired with its own pack.
`Pack` keeps its card-agnostic shape (resolved entries and unresolved refs), and its doc comment says it holds the refs of one card, or of any card-less caller, rather than a batch.
`Index.KickstartPack` now returns `([]CardPack, error)`, one `CardPack` per card in the order of the cards it is given; its doc comment says so, that a plan whose language opts out of glyphs yields nil, and that on an `ErrQuarryUnavailable` error every card lists its selected member glyphs unresolved with the error.
`planindex` takes no quarry import.

`selectPackRefs` takes one card and returns that card's refs in today's selection order (its groups in body order, then its `Uses:`), each spelling once within the card; the selection rule itself is unchanged: Edit, Delete and Move refs, a Rename pair's old side, `Uses:`, and any `plan:` handle, with plain paths, self glyphs, non-handle Create targets and a non-handle Rename new side left out.
`KickstartPack` returns `kickstartPack(plan, cards, resolve)` with `resolve` a closure calling `Resolve(worktreeRoot, targets)`; the body moves into `kickstartPack`, whose `resolve` parameter is the quarry boundary a test stubs.
`kickstartPack` returns nil for a plan whose language opts out of glyphs.
Otherwise it selects each card's refs, collects the batch's distinct member refs (parsed, non-self glyphs) in first-seen order, and calls `resolve` once over that distinct list, never when the list is empty.
It then builds a non-nil slice of one `CardPack` per card: a `plan:` handle is unresolved with `packHandleReason`; a member ref is unresolved with the resolve error's text when `Resolve` failed, and otherwise filed through `appendPackAnswer` from its answer in the one call.
A ref two cards name therefore appears under both, while quarry is called once.
On a resolve error it returns the degraded slice beside the error, as today, so begin-batch never refuses.
`index.KickstartPack` forwards with the new result type.

**Rendering (D2).**
`cardPointerPath` returns `planDirDisplay + "/" + filepath.Base(c.SourcePath)`, and `renderCardPointers` renders each bullet as `` - `<cardPointerPath>` `` through it, so the pointer list's bytes are unchanged (`TestPlanDirDisplay` keeps pinning both geometries).
`RenderForkPrompt` takes `pack []planindex.CardPack`: nil renders the prompt without a pack, exactly as a nil `*planindex.Pack` did; a non-nil slice is pack-on.
It computes `masterPlanDirDisplay(promptWorktreeRoot, planDir)` once, feeds it to both `renderCardPointers` and `renderPackSection`, and calls `renderLocate(stencilsDir, specsDir, pack != nil)`.
`renderLocate` takes `packOn bool` in place of the pack pointer, with unchanged output; `RenderRecoveryPrompt` passes `false`.

`renderPackSection` reads `webster-fragment-pack-section`, whose one marker is now `pack_cards`, and fills it with one `renderCardPack` result per `CardPack`, in slice order, joined by a blank line; each card's heading path is `cardPointerPath(cp.Card, planDirDisplay)`, so it equals that card's bullet in the card-pointer list under hub and standalone geometry alike.
`renderCardPack` renders one card's subsection from the new stencil `webster-fragment-pack-card`, whose text is a heading line `` ### `{{.card_path}}` ``, a blank line, then `{{.pack_entries}}{{.pack_unresolved}}`:
- The entries keep today's per-pack rule applied to the card's own pack: a card that names a member glyph (any resolved entry, or an unresolved ref that is not a `plan:` handle) lists one `` - `<glyph>` `` bullet per resolved entry with one `` - `<file>:<start>-<end>` — <signature> `` sub-bullet per declaration, or `none` when none of its member glyphs resolved; a card that names no member glyph gets the `webster-fragment-pack-empty` line.
- `pack_unresolved` is optional: a card with unresolved refs fills it with a blank line followed by the new stencil `webster-fragment-pack-unresolved` (a `#### Unresolved` heading, one sentence saying these refs of this card did not resolve, each with its reason, then `{{.pack_unresolved}}`) filled with one `` - `<ref>` — <reason> `` line per unresolved ref; a card with none fills it empty, so no unresolved list renders.
There is no batch-level unresolved list.
`webster-fragment-pack-section` keeps its `## Kick-start pack` heading and says that each card of this batch has a subsection under a heading that is that card's path in the card list above, with each entry's glyph and one line per declaration giving its file, line span and signature, and the card's unresolved refs after its entries.
`webster-fragment-pack-empty` reads `no member glyphs on this card`.
Each new or changed stencil's leading banner says who reads it, what fills its markers and where the result goes; `stencils.go` embeds the two new stencils with doc comments in the same style, registers them in `entries` right after `webster-fragment-pack-section`, and the `WebsterFragmentPackSection` and `WebsterFragmentPackEmpty` doc comments say per card.
`.gitattributes` gets the `text eol=lf` rows for the two new stencils.

**Log line (D6).**
`buildKickstartPack` returns `[]planindex.CardPack` (nil when off or for a language that opts out of glyphs).
When it re-parses the plan, it builds the card list by walking `batch.Cards` and taking each one's freshly parsed card by number, so the pack's cards follow the batch's declared order, the order `renderCardPointers` lists them in, rather than the plan's.
Its `webster: kickstart pack` Info line keeps its keys, and `resolved` and `unresolved` count distinct refs across the batch: the distinct `Glyph` values over every card's `Resolved`, and the distinct `Ref` values over every card's `Unresolved`, so a ref listed under two cards counts once.
`BeginBatch` passes the slice on to `RenderForkPrompt`; its call text is unchanged.

**Docs.**
`internal/websterengine/doc.go`'s `# the kick-start pack` section says the pack is grouped per card from one resolve call, that each card's subsection sits under the card's pointer path with its unresolved list only when it has unresolved refs, the per-card empty line and `none`, and that the log's resolved and unresolved count distinct refs across the batch.
`internal/planglyph/doc.go`'s KickstartPack sentence says it returns the pack card by card from one resolve call.
The `renderPackSection`, `RenderForkPrompt` and `buildKickstartPack` doc comments describe the per-card slice.

**Tests.**
`TestKickstartPack`'s want becomes `[]planindex.CardPack`, compared per card by card number and `Pack`:
- the existing rows keep their packs under their one card;
- the row "a ref repeated across cards appears once at its first position" becomes "a ref two cards name appears under both, once per card": card 1 edits `sub#Foo` and `sub#Bar` and uses `sub#Foo`, card 2 edits `sub#Baz` and `sub#Bar` and uses `sub#Foo`; card 1 wants Foo, Bar, card 2 wants Baz, Bar, Foo, each with the same declarations;
- the language-opts-out row wants nil;
- the missing-root subtest gets a second card that also edits `sub#Foo`, and wants both cards to list `sub#Foo` unresolved with `err.Error()`, card 1 also listing `plan:sub#Made` with the handle reason, and nothing resolved;
- a new subtest "the batch's distinct member refs are resolved in one call" calls `kickstartPack` on the two cards of the repeated-ref row with a stub `resolve` that records each call's targets and answers each target `quarry.StatusFound` with one symbol; it wants exactly one call, with targets `sub#Foo`, `sub#Bar`, `sub#Baz` in that order, and `sub#Foo` resolved under both cards; it fails if resolution runs per card or repeats a ref.

`fakeIndex`'s `pack` field becomes `pack func(cards []planparser.Card) []planindex.CardPack`; when set, `fakeIndex.KickstartPack` returns `pack(cards)` with `packErr`, so a row can pair its canned pack with the batch's real cards.
`TestBeginBatch_KickstartPack`:
- a new case field `cardHeadings []string` lists card file base names whose heading `` ### `<display>/<base>` `` the prompt must hold, where `<display>` is the same display the prompt's card-pointer bullets use; the row's plan directory is a `t.TempDir()` sibling of the fixture worktree (`newBeginFixture`'s own `t.TempDir()`), so `masterPlanDirDisplay` gives the absolute plan directory and this is standalone geometry; the loop asserts both the pointer bullet `` - `<display>/<base>` `` and the heading, so a fixture change that moves the plan under the worktree fails on the pointer, naming the cause; the switch-on row wants `01-json-flag.md`;
- the degrade row's `prepare` sets `pack` to a func returning card 1 with `internal/foo#Baz` unresolved as today, and wants the `#### Unresolved` heading and the unresolved line;
- a new case field `oneBatch` gives card 2 an `Edit` group on `internal/foo#Baz` (with an `ImpactSummary`) and puts both cards in batch 1;
- a new row "a glyph two cards of the batch name is counted once" sets `oneBatch` on the real index, wants both cards' headings and the log `batch=1 kickstart_pack=on resolved=1 unresolved=0`, which fails if `resolved` counts listings;
- a new row "an unresolved ref two cards list is counted once" sets `oneBatch` and a `prepare` whose `pack` func returns both of the given cards, each with `internal/foo#Baz` unresolved, and `packErr` wrapping `ErrQuarryUnavailable`; it wants the log `batch=1 kickstart_pack=on resolved=0 unresolved=1`, which fails if `unresolved` counts listings.

`renderFork` takes `[]planindex.CardPack`; `TestRenderPrompts_FrictionDirective` and `TestRenderPrompts_ParentDirective` keep passing nil.
`TestRenderPrompts`'s pack rows become per card, under hub geometry (`_lyx/plan`):
- the multipart row renders a two-card batch (`alpha` and a second card `beta` from `cardWithSourcePath`), card 1 with `internal/foo#Bar` (two declarations) and `internal/foo#Baz`, card 2 with `internal/foo#Baz`; it wants in order the pointer bullets, then `` ### `_lyx/plan/01-alpha.md` ``, Bar's and Baz's lines, `` ### `_lyx/plan/02-beta.md` ``, Baz's line, and not `#### Unresolved`; its `notContains` needle `no member glyphs on this batch's cards` becomes `no member glyphs on this card`, so a card that names member glyphs is still asserted not to render the per-card empty line;
- the unresolved row wants in order the heading, Bar's line, `#### Unresolved` and both unresolved lines;
- the empty-pack row passes one card with an empty `Pack` and wants `no member glyphs on this card` and not `#### Unresolved`;
- the `plan:`-handles-only row wants in order `no member glyphs on this card`, `#### Unresolved` and the handle line;
- a new row "a card whose member glyphs none resolved renders none then its unresolved list" passes `internal/foo#Gone` unresolved alone and wants in order the heading, `\nnone\n`, `#### Unresolved` and the `internal/foo#Gone` line.
Standalone geometry is reached through `TestBeginBatch_KickstartPack`'s `cardHeadings`, and hub geometry through these rows.

`TestPlanDirDisplay`'s `//testtiming:keep` reason no longer says the covering render and begin-batch tests run hub geometry only: it says the test pins the plan-directory display and the pointer bullets of both geometries byte for byte, while the covering tests each reach one geometry.

`contracts/stencils`: `TestStencils_TopLevelMarkers` rows: `webster-fragment-pack-section` -> `pack_cards`, `webster-fragment-pack-card` -> `card_path`, `pack_entries`, `pack_unresolved`, and `webster-fragment-pack-unresolved` -> `pack_unresolved`.
`wordingClaims`' `webster-fragment-pack-empty.md` claim becomes `no member glyphs on this card`, with a `wantNone` on `this batch's cards`; new entries claim the card stencil's `` ### `{{.card_path}}` `` heading, the unresolved stencil's `#### Unresolved` heading, and the section stencil's sentence that each card's subsection sits under its path in the card list.

**ImpactSummary:** Changes the `planindex.Index.KickstartPack` contract, whose implementers are `planglyph`'s index and websterengine's `fakeIndex`, and whose only production consumer is `buildKickstartPack`; `RenderForkPrompt` has one production caller (`BeginBatch`) and the `renderFork` test helper; the recovery prompt's bytes are unchanged.

**Commit:** 2: group the kick-start pack per card
