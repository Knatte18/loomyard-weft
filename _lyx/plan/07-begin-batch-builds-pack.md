# Card 7 — begin-batch builds the pack into the fork prompt

**Create:**
- `plan:internal/websterengine#buildKickstartPack` -> `func buildKickstartPack(deps BeginDeps, index planindex.Index, batch batcher.Batch, number int) (*planindex.Pack, error)`
- `plan:internal/websterengine_test#fakeIndex.KickstartPack` -> `func (i *fakeIndex) KickstartPack(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string) (planindex.Pack, error)`
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
- `internal/websterengine_test#seedRewritingPlanDir`
- `internal/websterengine_test#captureLogs`

**Intent:**
Decisions D3, D5, D6 and D9: each `begin-batch` builds the batch's kick-start pack and renders it into the fork prompt file only.

`BeginBatch` calls `buildKickstartPack` immediately before `RenderForkPrompt`, which comes after the dispatch re-resolution (`ValidateDispatch`) and its fingerprint re-stamp, and passes the result to `RenderForkPrompt`.
`buildKickstartPack` behaves as follows:
- With `deps.Config.KickstartPack` false, it returns a nil pack.
- Otherwise it parses the plan afresh from `deps.Plan.Dir` with `planparser.ParsePlan`.
  The re-resolution may have canonicalized handles on disk, so the pack's spellings must match the card files the fork reads.
  A parse error is returned, and `BeginBatch` wraps it as a transient error naming `lyx webster begin-batch N` as the way forward, as it does for the card-hash read.
- When the fresh plan has `language: none`, it returns a nil pack, so the prompt renders as with the switch off.
- Otherwise it takes the fresh plan's cards whose numbers are the batch's.
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

`TestBeginBatch_KickstartPack` is a new table test in `internal/websterengine/beginbatch_test.go`.
It runs over `newBeginFixture` with `captureLogs`, at a verbosity that keeps Info lines, so it does not call `t.Parallel`.
Rows:
- Switch on, with the card's `Edit` member glyph written into the fixture worktree: the prompt file carries the pack section with the glyph, file, span and signature.
  `BeginResult.Advisories` and every other `BeginResult` field carry no pack text.
  The D9 line reads `on` with the counts.
- Switch off: the prompt carries the quarry fragment and no pack heading, and the D9 line reads `off`.
- A `language: none` plan with the switch on renders as switch off, and the D9 line reads `language-none`.
- A `fakeIndex.pack` degraded pack with an `ErrQuarryUnavailable` `packErr`: `begin-batch` succeeds, the prompt lists every entry under the unresolved heading, and a Warn line names the error.
- With the switch on over `seedRewritingPlanDir`, whose handles the re-resolution canonicalizes: the pack lists the canonical `plan:` spelling now on disk, not the draft.

Raise the `internal/websterengine` row in `cmd/lyx/testdata/test-budget.yaml` by one more.
In `internal/websterengine/doc.go`, add a section on the kick-start pack:
- what it holds (D4)
- where and when `begin-batch` builds it, and the degraded and `language: none` cases (D5)
- that it reaches only the fork and never Master or the envelope (D6)
- the two locate fragments and the pack section (D7, D8)
- the D9 trace line
- the `kickstart_pack` switch

**ImpactSummary:** `BeginBatch` now builds and renders the pack when `kickstart_pack` is on, which is the template default, so every Webster fork prompt changes; its envelope, refusals and state writes are unchanged, and the switch set to `false` restores today's prompt.
