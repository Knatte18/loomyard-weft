# Card 8 — begin-batch builds the pack into the fork prompt

**Create:**
- `plan:internal/websterengine#buildKickstartPack` -> `func buildKickstartPack(deps BeginDeps, index planindex.Index, batch batcher.Batch, number int) *planindex.Pack`
- `plan:internal/websterengine_test#fakeIndex.KickstartPack` -> `func (i *fakeIndex) KickstartPack(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string) (planindex.Pack, error)`
- `plan:internal/websterengine_test#fakeIndex.ValidateDispatch` -> `func (i *fakeIndex) ValidateDispatch(plan *planparser.Plan, worktreeRoot string, completed, forthcoming []planparser.Card) ([]planindex.Finding, error)`
- `plan:internal/websterengine_test#TestBeginBatch_KickstartPack` -> `func TestBeginBatch_KickstartPack(t *testing.T)`

**Edit:**
- `internal/websterengine#BeginBatch`
- `internal/websterengine/doc.go`
- `internal/websterengine_test#fakeIndex`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/websterengine#Config`
- `internal/websterengine#RenderForkPrompt`
- `internal/websterengine_test#newBeginFixture`
- `internal/websterengine_test#mustFingerprint`
- `internal/websterengine_test#writeWorktreeFile`
- `internal/websterengine_test#captureLogs`

**Intent:**
Decisions D3, D5, D6 and D9: each `begin-batch` builds the batch's kick-start pack and renders it into the fork prompt file only.

`BeginBatch` calls `buildKickstartPack` immediately before `RenderForkPrompt`, which comes after the dispatch re-resolution (`ValidateDispatch`) and its fingerprint re-stamp, and passes the result to `RenderForkPrompt`.
`buildKickstartPack` behaves as follows:
- With `deps.Config.KickstartPack` false, it returns a nil pack.
- Otherwise it parses the plan afresh from `deps.Plan.Dir` with `planparser.ParsePlan`.
  The re-resolution may have canonicalized handles on disk, so the pack's spellings must match the card files the fork reads.
  A parse error degrades the pack and never refuses, as D5 has a failed resolve degrade: it logs at Warn with the error and builds the pack from `deps.Plan` and `batch.Cards` instead.
  That pack's member glyphs resolve as usual, and only a `plan:` handle the re-resolution canonicalized may keep its draft spelling.
  `buildKickstartPack` therefore returns no error, and `BeginBatch` gains no refusal.
- When the plan the pack is built from has `language: none`, it returns a nil pack, so the prompt renders as with the switch off.
- Otherwise it takes that plan's cards whose numbers are the batch's.
  It calls `index.KickstartPack` with them and `deps.Geom.WorktreeRoot`, the told root (`PATTERN-told-geometry`).
- On a `KickstartPack` error, it logs at Warn with the error and keeps the degraded pack the index returned.
  `begin-batch` does not refuse.
- In every case it logs one Info line, `webster: kickstart pack`, with keys `batch`, `kickstart_pack` (`on`, `off` or `language-none`), `resolved` and `unresolved` (entry counts, `0` when no pack is built).

The pack reaches only the fork prompt file under `PromptsDir` (D6).
`BeginResult` gains no field, and unresolved refs never join `Advisories`.
A `no_report` re-fork reuses the prompt file `begin-batch` wrote, pack included.
Each `begin-batch` reads `kickstart_pack` from its own `Config`, so a flip applies from the next call on.

The switch lets the fork skip its own quarry lookup only for entries the pack resolved.
An unresolved member glyph, up to every one of a degraded pack's, keeps today's locate path through the pack fragment's instruction.
`RenderRecoveryPrompt` never takes a pack.

`fakeIndex` gains `pack *planindex.Pack` and `packErr error`.
When `pack` is set, its new `KickstartPack` override returns `*pack, packErr`.
Otherwise it forwards to the embedded real index.
`fakeIndex` also gains `breakPlanAfterValidate bool`.
Its new `ValidateDispatch` override forwards to the embedded index and then, when the field is set, appends to the plan's `00-overview.md` Card Index a line for card 3, which has no card file.
`restampFingerprint` accepts that overview, since it keeps its frontmatter and its `## Card Index` and the batch's own card files still hash.
`planparser.ParsePlan` refuses it, since `parseCardFile` finds no file for card 3, so the fresh parse in `buildKickstartPack` fails after the re-resolution passed.

`TestBeginBatch_KickstartPack` is a new table test in `internal/websterengine/beginbatch_test.go`.
It runs over `newBeginFixture` with `captureLogs`, at a verbosity that keeps Info lines, so it does not call `t.Parallel`.
Each row writes its own plan with `plankit.Write` into a `t.TempDir()`, because `buildKickstartPack` reads the plan on disk and `seedPlanDir`'s cards carry no member glyph.
It parses that plan into `fx.Deps.Plan`, points `fx.Deps.Geom.PlanDir` at it and sets `fx.Deps.State.PlanFingerprint` with `mustFingerprint`, as `TestBeginBatch_RestampsFingerprintEvenWhenPlanDrifts` does.
The plan's cards 1 and 2 match the fixture's batches, and card 2 is a `Prosa` card on `base.txt`.
Card 1 carries an `ImpactSummary`, since its `Edit` group makes `ValidateFormat`'s `card-missing-field` require one, and every ref resolves against the fixture worktree, so `ValidateDispatch` reports nothing blocking.
Rows:
- Switch on, card 1 an `Edit` card on a member glyph whose declaration the row writes into the fixture worktree with `writeWorktreeFile`: the prompt file carries the pack section with the glyph, file, span and signature.
  `BeginResult.Advisories` and every other `BeginResult` field carry no pack text.
  The D9 line reads `on` with the counts.
- Switch off over the same plan: the prompt carries the quarry fragment and no pack heading, and the D9 line reads `off`.
- The same plan written with `language: none` and the switch on: the prompt renders as switch off, and the D9 line reads `language-none`.
- A `fakeIndex.pack` degraded pack with an `ErrQuarryUnavailable` `packErr`: `begin-batch` succeeds, the prompt lists every entry under the unresolved heading, and a Warn line names the error.
- Switch on, card 1 declaring the draft handle `plan:internal/foo#Barr` -> `func Bar()` beside an `Edit` member glyph that resolves, as `seedRewritingPlanDir`'s card 1 does without its unresolvable card 2: the re-resolution canonicalizes the handle on disk, and the pack lists the canonical `plan:internal/foo#Bar` spelling, not the draft.
- The same plan with `breakPlanAfterValidate` set: `begin-batch` succeeds, a Warn line names the parse error, the pack still lists the resolved member glyph, and the D9 line reads `on`.

Raise the `internal/websterengine` row in `cmd/lyx/testdata/test-budget.yaml` by one more.
In `internal/websterengine/doc.go`, add a section on the kick-start pack:
- what it holds (D4)
- where and when `begin-batch` builds it, and the degraded and `language: none` cases (D5)
- that it reaches only the fork and never Master or the envelope (D6)
- the two locate fragments and the pack section (D7, D8)
- the D9 trace line
- the `kickstart_pack` switch

**ImpactSummary:** `BeginBatch` now builds and renders the pack when `kickstart_pack` is on, which is the template default, so every Webster fork prompt changes; its envelope, refusals and state writes are unchanged, and the switch set to `false` restores today's prompt.
