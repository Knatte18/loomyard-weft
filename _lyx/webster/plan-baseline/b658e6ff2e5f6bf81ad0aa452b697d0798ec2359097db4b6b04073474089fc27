# Card 4 — implementer-verify-tiers

**Edit:**
- `internal/websterengine#RenderForkPrompt` -> `func RenderForkPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, reportPath, planDir, promptWorktreeRoot, stencilsDir, specsDir string, verifyTiers []string, outcomePath, summaryPath string, selfFixCap int, notePath, packPath string, layout PackLayout, pack []planindex.CardPack) (prompt, packFile []byte, err error)`
- `internal/websterengine#RenderRecoveryPrompt` -> `func RenderRecoveryPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, failureDigest, uncommittedPaths, reportPath, repoRoot, planDir, promptWorktreeRoot, stencilsDir, specsDir string, verifyTiers []string, selfFixCap int, notePath, parentName string) ([]byte, error)`
- `internal/websterengine#BeginDeps`
- `internal/websterengine#RecoverDeps`
- `internal/websterengine#BeginBatch`
- `internal/websterengine#recoverSpawn`
- `internal/websterengine/doc.go`
- `internal/websterengine_test#forkTemplateMarkerValues`
- `internal/websterengine_test#renderForkWithPack` -> `func renderForkWithPack(batch batcher.Batch, root, stencilsDir, specsDir, prevDigest string, selfFixCap int, notePath string, layout websterengine.PackLayout, pack []planindex.CardPack, verifyTiers []string) (prompt, packFile string, err error)`
- `internal/websterengine_test#renderRecoveryWithUncommitted` -> `func renderRecoveryWithUncommitted(batch batcher.Batch, repoRoot, anchorRoot, stencilsDir, specsDir, failureDigest, uncommittedPaths, notePath, parent string, verifyTiers []string) (string, error)`
- `internal/websterengine_test#renderFork`
- `internal/websterengine_test#renderForkPromptAndPack`
- `internal/websterengine_test#renderRecovery`
- `internal/websterengine_test#TestRenderPrompts_ParentDirective`
- `internal/websterengine_test#TestTemplates_FillRequiresEveryRequiredMarker`
- `internal/websterengine_test#TestRenderPrompts`
- `internal/websterengine_test#TestBeginBatch_PromptFile`
- `internal/websterengine_test#TestRecoverBatch_UncommittedPathsPrompt`
- `internal/webstercli#websterCLI`
- `internal/webstercli#websterCLI.wireHub`
- `internal/webstercli#websterCLI.beginBatchCmd`
- `internal/webstercli#websterCLI.recoverDeps`
- `internal/webstercli#TestWireHub_LanguageSkills`
- `contracts/stencils/webster/webster-body-implementer.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/webstercli#websterCLI.beginDeps` -> `func (c *websterCLI) beginDeps(plan *planparser.Plan, batches []batcher.Batch, st *websterengine.State) websterengine.BeginDeps`
- `plan:contracts/stencils#TestHubGenericStencils_NameNoGoTiers` -> `func TestHubGenericStencils_NameNoGoTiers(t *testing.T)`

**Uses:**
- `internal/languageconfig#VerifyTiersValue`

**Intent:** The implementer body, shared by the in-session fork and the recovery strand, renders the hub's `verify_tiers` in its raw-run section instead of naming raw `go` runs (D6).

- `RenderForkPrompt` and `RenderRecoveryPrompt` take the told `verifyTiers` after `specsDir` and fill a required `verify_tiers` marker with `languageconfig.VerifyTiersValue(verifyTiers)`; neither adds it to its optional list.
  Their doc comments name the parameter and its `none` rendering, and `render.go`'s file header, which names `specsDir` as the extra required parameter both renderers take after `stencilsDir`, names `verifyTiers` after it the same way.
- `BeginDeps` and `RecoverDeps` gain `VerifyTiers []string` (empty for none); `BeginBatch` and `recoverSpawn` pass `deps.VerifyTiers`.
- `websterCLI` gains a `verifyTiers []string` field beside `languageSkills`: `wireHub` stores the loaded `Language.VerifyTiers`, standalone leaves it empty.
  `beginBatchCmd`'s inline `BeginDeps` literal moves into `beginDeps`, built like `recoverDeps` and carrying `VerifyTiers`, so a test reaches it; `recoverDeps` carries it too.
- `webster-body-implementer.md`'s `## Slot-busy exits and raw go runs` section is retitled to name raw test runs (for example `## Slot-busy exits and raw test runs`).
  The slot-busy paragraph stays as is, `75` included.
  The raw-run paragraph says: a raw test run of the hub's language is allowed on the batch's listed packages only; the hub's verify tiers follow as `{{.verify_tiers}}` on its own line, and a tier the list says is compiled rather than run is compiled, never run, as the list says; with `none`, a raw run is still limited to the listed packages and to what the sentences below allow.
  The guard-describing sentence stays in language-neutral form: a module-wide, `tmux`-tier or `llm`-tier run, which the shell guard refuses raw, goes through `lyx gate test`.
  The sentence that a `lyx gate test` step runs as written, whatever its `--tags`, stays, so the list's compile-only rule governs raw runs and runs no gate step names, never a rendered card or batch gate step.
  The gate-describing sentence stays: a test under the `tmux` or `llm` tag that no gate step names is compiled, never run, through `lyx gate test --tags <tag> <packages> -- -run '^$'`, and the list's compile-only rule adds tiers beyond those two.
  The failure and report sentences `webster-fork-stencil-fixes` cites are not edited.
  A tier the list calls compiled is compiled, never run, by the implementer's raw runs only, while every rendered card or batch gate step still runs as written, which bounds it.
- `doc.go`'s sentence on the master's and recovery strand's language skills (`RunDeps.LanguageSkills`, `RecoverDeps.LanguageSkills`) gains one beside it: the fork and recovery prompts render the hub's verify tiers from `BeginDeps.VerifyTiers` and `RecoverDeps.VerifyTiers`, `none` for an empty list.

`TestHubGenericStencils_NameNoGoTiers` is the one scan behind the acceptance criterion that a hub with an empty list gets no Go tier commands: over the registered default text of `loom-template-plan`, `loom-rubric-plan-review` and `webster-body-implementer`, it fails on any of `go build`, `go vet`, `go test`, `-tags tmux`, `-tags llm` and `go -C`, shaped like `TestTwillStencils_HostNeutral` and carrying `//lyx:guard`.
With an empty list the only text these prompts add is `none`, so the scan over the stencil text covers the rendered prompts.
The sentences the Out list keeps (the shell guard, `lyx gate test`, the `tmux`/`llm` compile-only sentence) spell none of those strings, so the scan needs no exemption.
It lands on this card because Cards 2 and 3 have already edited the other two stencils; the `contracts/stencils` row in `cmd/lyx/testdata/test-budget.yaml` rises by one.

Tests:

- `forkTemplateMarkerValues` gains a `verify_tiers` placeholder (the recovery map inherits it), and `TestTemplates_FillRequiresEveryRequiredMarker` lists `verify_tiers` among the fork's and the recovery's required markers.
- `renderForkWithPack` and `renderRecoveryWithUncommitted` gain a `verifyTiers []string` parameter that they pass to `RenderForkPrompt` and `RenderRecoveryPrompt`; `renderFork`, `renderRecovery`, `renderForkPromptAndPack` and `TestRenderPrompts_ParentDirective`'s direct calls pass nil.
- `TestRenderPrompts` gains rows in its `cases` table, under `testLayout`: the fork and the recovery prompt rendered with two tiers, one holding `{{.x}}`, through direct calls to `renderForkWithPack` and `renderRecoveryWithUncommitted` with the tiers, each carry both bullets consecutively, in order and verbatim;
  the fork and the recovery prompt rendered with nil tiers each carry `none` after the list sentence and hold no `- ` line with empty content.
- `TestBeginBatch_PromptFile` gains a row whose `prepare` sets `fx.Deps.VerifyTiers` and whose `want` holds the tier bullet, read from the prompt file `BeginBatch` writes.
- `TestRecoverBatch_UncommittedPathsPrompt` sets `RecoverDeps.VerifyTiers` and asserts the recovery prompt it spawns carries the tier bullet.
- `TestWireHub_LanguageSkills`'s listed row writes tiers beside the skills and asserts `c.verifyTiers`, `c.beginDeps(nil, nil, nil).VerifyTiers` and `c.recoverDeps(nil, nil, nil).VerifyTiers` equal them.

**ImpactSummary:** Two renderers re-signed, each with one production caller (`BeginBatch`, `recoverSpawn`) and test helpers; two deps structs and the webster CLI gain an additive field, with `beginBatchCmd`'s deps literal moved into a helper; the implementer body gains one required marker filled at both render sites, and one guard scan pins the three hub-generic stencils free of Go tier commands.
