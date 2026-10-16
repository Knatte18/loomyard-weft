# Card 13 — implementer-rename-verify-markers

**Edit:**
- `internal/websterengine#RenderForkPrompt` -> `func RenderForkPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, reportPath, planDir, promptWorktreeRoot, stencilsDir, specsDir string, verifyTiers, renameTools []string, planVerify, outcomePath, summaryPath string, selfFixCap int, notePath, packPath string, layout PackLayout, pack []planindex.CardPack) (prompt, packFile []byte, err error)`
- `internal/websterengine#RenderRecoveryPrompt` -> `func RenderRecoveryPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, failureDigest, uncommittedPaths, reportPath, repoRoot, planDir, promptWorktreeRoot, stencilsDir, specsDir string, verifyTiers, renameTools, patternChanges []string, planVerify string, selfFixCap int, notePath, parentName string) ([]byte, error)`
- `internal/websterengine#BeginDeps`
- `internal/websterengine#RecoverDeps`
- `internal/websterengine#BeginBatch`
- `internal/websterengine#recoverSpawn`
- `internal/websterengine/doc.go`
- `internal/webstercli#websterCLI`
- `internal/webstercli#websterCLI.wireHub`
- `internal/webstercli#websterCLI.beginDeps`
- `internal/webstercli#websterCLI.recoverDeps`
- `contracts/stencils/webster/webster-body-implementer.md`
- `internal/websterengine_test#renderForkWithPack` -> `func renderForkWithPack(batch batcher.Batch, root, stencilsDir, specsDir, prevDigest string, selfFixCap int, notePath string, layout websterengine.PackLayout, pack []planindex.CardPack, verifyTiers, renameTools []string, planVerify string) (prompt, packFile string, err error)`
- `internal/websterengine_test#renderRecoveryWithUncommitted` -> `func renderRecoveryWithUncommitted(batch batcher.Batch, repoRoot, anchorRoot, stencilsDir, specsDir, failureDigest, uncommittedPaths, notePath, parent string, verifyTiers, renameTools, patternChanges []string, planVerify string) (string, error)`
- `internal/websterengine_test#renderFork`
- `internal/websterengine_test#renderForkPromptAndPack`
- `internal/websterengine_test#renderRecovery`
- `internal/websterengine_test#forkTemplateMarkerValues`
- `internal/websterengine_test#TestTemplates_FillRequiresEveryRequiredMarker`
- `internal/websterengine_test#TestRenderPrompts`
- `internal/websterengine_test#TestRenderPrompts_FrictionDirective`
- `internal/websterengine_test#TestRenderPrompts_ParentDirective`
- `internal/websterengine_test#TestBeginBatch_PromptFile`
- `internal/websterengine_test#TestRecoverSpawnOrAttach`
- `internal/webstercli#TestWireHub_LanguageSkills`

**Uses:**
- `internal/languageconfig#VerifyTiersValue`

**Intent:**
D1.3, D2.6, D2.7 and the implementer half of D2.2: the shared implementer body names the hub's rename tool, knows whether the plan declares a `## verify:` section, and points at the departure trailer.

- `BeginDeps` and `RecoverDeps` gain `RenameTools []string`; `websterCLI` gains `renameTools`, which `wireHub` fills from `language.yaml` beside `verifyTiers` and `beginDeps` and `recoverDeps` pass; a standalone run leaves it empty.
- `RenderForkPrompt` and `RenderRecoveryPrompt` gain `renameTools`, filled into a new required `rename_tools` marker with `languageconfig.VerifyTiersValue` (bullets, or `none`), and `planVerify`, filled into a new required `plan_verify` marker as the command, or `none` when empty.
  `BeginBatch` and `recoverSpawn` pass `deps.RenameTools` and `deps.Plan.Verify`.
- `webster-body-implementer.md`:
  - step 2 gains the rename sentence: for each symbol pair of a `Rename` card, run the hub's rename tool listed under `{{.rename_tools}}` (rendered as its own bullet block), then the grep pass for zero old-name references and the updated doc-comment opening; a file pair is `git mv` plus surgical Edits.
    It names the tool and restates no edit rule: the exception lives in the edit directive alone (`PATTERN-edit-directive`, card 17).
  - a new paragraph after step 4 says that a card departing from its own text because a PATTERN entry forbids the card's way records the departure as a `Pattern-Departure:` trailer in that card's commit, in the shape `{{.specs_dir}}/loom/loom-plan-spec.md`'s `## Pattern-Departure trailer` section defines, which it points at and does not restate.
  - the self-fix section's "name it in your final reply to Master and move on" becomes: a failure the card did not cause is left to the plan-level verify gate.
    It renders the plan's verify command, `{{.plan_verify}}`, and says that when it is a command, a batch gate step that fails only on failures the batch's cards did not cause, by the two-pronged test above, does not make the report `FAILED`, and that when it is `none` such a failure stays `FAILED`.
    It states the bound: a misjudged failure passes the batch and is caught by the plan-level verify gate, which runs that command and spawns a fixer when it fails, so the catch covers what the command runs.
  - the `status` paragraph says the same, so the self-fix section and the status rule agree, and the report paragraph says `record-batch` reads the two fields, not Master.
- `doc.go` names the two new markers and where their values come from.

Tests:
`renderForkWithPack` and `renderRecoveryWithUncommitted` gain `renameTools` and `planVerify` arguments; `renderFork`, `renderForkPromptAndPack` and `renderRecovery` pass nil and an empty command through, and the direct `RenderForkPrompt` and `RenderRecoveryPrompt` calls and helper calls in `TestRenderPrompts`, `TestRenderPrompts_FrictionDirective` and `TestRenderPrompts_ParentDirective` pass the same; `forkTemplateMarkerValues` fills both markers and `TestTemplates_FillRequiresEveryRequiredMarker` lists both as required for the fork and the recovery template.
`TestRenderPrompts` gains rows: a fork and a recovery prompt rendered with two rename tools carry them as bullets, and with none carry `none` under the rename sentence; a prompt rendered with a verify command carries it, and with an empty one carries `none`.
`TestBeginBatch_PromptFile` gains a row whose deps carry a rename tool and whose plan declares a `## verify:` command: the written prompt file carries the tool's bullet and the command, so a `BeginBatch` that passed neither would render `none` and fail the row.
`TestRecoverSpawnOrAttach` gains the same row for a fresh recovery spawn, asserted on `fx.Engine.LastPrompt`.
`TestWireHub_LanguageSkills`' listing row adds a `rename_tools` list and asserts it reaches `c.renameTools`, `beginDeps` and `recoverDeps`.

**ImpactSummary:** two re-signed renderers, each with one production caller (`BeginBatch`, `recoverSpawn`) and one test helper; the body gains two required markers both callers fill.
