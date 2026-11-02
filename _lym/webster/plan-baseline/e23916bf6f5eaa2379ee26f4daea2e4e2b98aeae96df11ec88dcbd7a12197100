# Card 14 — webster-fork-card-fixer-text

**Create:**
- `contracts/stencils/webster/webster-fragment-card-fixer.md`
- `internal/webstercli#cardFixerOn`
- `internal/webstercli#TestCardFixerOn`
- `contracts/stencils#WebsterFragmentCardFixer`

**Edit:**
- `contracts/stencils/webster/webster-body-implementer.md`
- `internal/websterengine#RenderForkPrompt` -> `func RenderForkPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, reportPath, planDir, promptWorktreeRoot, stencilsDir, specsDir, gateTiers string, renameTools []string, outcomePath, summaryPath string, selfFixCap int, notePath string, cardFixer bool) ([]byte, error)`
- `internal/websterengine#RenderRecoveryPrompt` -> `func RenderRecoveryPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, failureDigest, uncommittedPaths, reportPath, repoRoot, planDir, promptWorktreeRoot, stencilsDir, specsDir, gateTiers string, renameTools, patternChanges []string, selfFixCap int, notePath, parentName string, cardFixer bool) ([]byte, error)`
- `internal/websterengine#BeginDeps`
- `internal/websterengine#RecoverDeps`
- `internal/websterengine#BeginBatch`
- `internal/websterengine#recoverSpawn`
- `internal/webstercli#websterCLI.beginDeps`
- `internal/webstercli#websterCLI.recoverDeps`
- `internal/websterengine_test#renderFork`
- `internal/websterengine_test#renderForkWithLists`
- `internal/websterengine_test#renderRecoveryWithUncommitted`
- `internal/websterengine_test#TestRenderPrompts`
- `internal/websterengine_test#TestRenderPrompts_ParentDirective`
- `contracts/stencils#TestStencils_TopLevelMarkers`

**Extends:**
- `contracts/stencils#entries`
- `internal/websterengine/doc.go`

**Uses:**
- `internal/webstercli#websterCLI.pairSlug`
- `internal/gateslot#GateKey`
- `internal/webstercli/doc.go`

**Intent:**
When the hub's `card` gate key carries a non-empty `fixer` and the worktree has a pair slug, the Webster fork's and recovery strand's prompt says a failed card gate call has already been through a cold fixer whose reply its report carries, and that the strand fixes what is left under its self-fix cap; otherwise the text is today's (D7).

- `webster-fragment-card-fixer.md` holds that paragraph; `webster-body-implementer.md` gains an optional `{{.card_gate_fixer}}` marker right after the card gate's step list, which `RenderForkPrompt` and `RenderRecoveryPrompt` fill with the fragment when `cardFixer` is true and leave empty otherwise, so the off text is today's byte for byte.
  The fragment also tells the strand to read the uncommitted diff the fixer left before it commits the card, since that read is what judges the card fixer's edits (D5).
- `BeginDeps` and `RecoverDeps` gain `CardFixer bool`; `BeginBatch` and `recoverSpawn` pass it.
- `cardFixerOn(gate, slug)` is true only when `slug` is non-empty and the gate config's `card` key exists with a non-empty `Fixer`; `websterCLI.beginDeps` and `websterCLI.recoverDeps` set `CardFixer` from it over the loaded gate config and `pairSlug()`.
- The fragment is embedded and registered with its marker row, and the body's row gains `card_gate_fixer`.
- `doc.go` names the card-gate fixer paragraph and when it renders.

Tests, tier0:
- `TestRenderPrompts` gains rows: the fork and the recovery prompt with `cardFixer` true carry the fragment's paragraph after the card gates, and with `false` carry none; the test helpers pass `false` as today's callers do.
- `TestRenderPrompts_ParentDirective` passes `false` and keeps its assertions.
- `TestCardFixerOn` rows: a `card` key with `fixer` and a slug is on; an empty slug, a `card` key with an empty `fixer` and a config with no `card` key are off.

**ImpactSummary:** two prompt renderers are re-signed with every caller in `internal/websterengine` and its tests; the rendered text changes only on a pair whose `card` key carries a fixer.
