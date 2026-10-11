---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/websterengine/recordbatch.go:849
    summary: a restamp failure after a successful bind returns before the bind trace is logged, a no-line case card 5 and doc.go do not list
---

Range: no live rework round exists (`_lyx/loom/rework/` is absent), so the range is `git diff $(git merge-base main HEAD)..HEAD`.
`go build`, `go vet` and `lyx gate test` over `planparser`, `planindex`, `planglyph`, `websterengine`, `contracts/stencils` and `cmd/lyx` are green.

Every focus question was settled against the code:

- `buildKickstartPack` returns no error and degrades with a Warn on a fresh-parse failure (keeping `deps.Plan` and `batch.Cards`) and on a `KickstartPack` error.
  It returns a nil pack for switch off and for `language: none`, and emits exactly one `webster: kickstart pack` Info line with keys `batch`, `kickstart_pack`, `resolved` and `unresolved` on every path.
- The pack reaches only `RenderForkPrompt`.
  `BeginResult` has no new field, no advisory carries pack text, and `RenderRecoveryPrompt` fills `locate` with the quarry fragment and never a pack.
  The nil-pack render is pinned byte-for-byte by the `TestRenderPrompts` row the card describes, and the quarry fragment holds the two former step-2 sentences verbatim.
- `internal/websterengine` imports neither `planglyph` nor quarry, and `planindex` imports only `errors`, `fmt` and `planparser`.
  The `quarry/glyph` dependency reached through `planparser` is not new.
- `cardOwnHandles` has no remaining reference outside `_lyx`; `planglyph.BindHandles` calls `planparser.OwnedHandles` and its comments and `doc.go` name it.
- Bind trace: both terminal callers set `Number`, the four reasons sit beside `postBatchChecks`, `language-none` and `delta-unavailable` replace the computed outcomes, and `TestRecordBatch_BindTrace` carries every row card 5 names.
- Test economy: new top-level tests are `TestKickstartPack`, `TestBeginBatch_KickstartPack` and `TestRecordBatch_BindTrace`, and the budget rows rose by one (`planglyph`) and two (`websterengine`), matching the cards.
  The `BindHandles` tests were extended rather than duplicated, and `OwnedHandles` is covered through them as card 4 states.
- Cards 6 and 7: `kickstart_pack` defaults to `true` in `template.yaml`, and the pack fragment keeps the `lyx quarry resolve` or grep path for an unresolved member glyph.

### [LOW] A restamp failure skips the bind trace

**Location:** `internal/websterengine/recordbatch.go:849`
**Issue:** `postBatchChecks` returns `rebaseErr` after `BindHandles` succeeded and before `logBindOutcomes` runs, so that pass logs no bind line although the bind and its `RewriteRefs` already landed.
Card 5 lists four no-line cases, and `doc.go` lists the same four as the passes that fail before binding; this fifth case is in neither.
**Fix:** Either log the outcomes before the restamp error return, or add the restamp failure to the card-5 no-line list in `doc.go`.
