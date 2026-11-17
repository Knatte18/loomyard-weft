```yaml
lens: code-tests
outcome: reported
```

Restating my task: the code-tests lens. For each behaviour the diff adds or changes, name the test row that would fail if it broke.

**Per behaviour: the test that pins it, or none**
- `ReadForEngine` writes the launch engine's copy in place, keeps the header and leaves plain includes alone: `TestReadForEngine` rows `ClaudeCopyStandsInsideTheLine` and `CodexCopyStandsInsideTheLine`.
- `ReadForEngine` refusals: rows `EngineWithNoCopyIsAMissingBlock`, `NoEngineIsRefused`, `TrimmedActionIsRefused` and `EngineBlockIncludingAnotherIsRefused`.
- An engine-block copy is read without its final newline: `TestReadSet` rows `EngineBlockReadsTheClaudeCopy` and `EngineBlockReadsTheCodexCopy`, whose wanted values carry no `\n`.
- `ReadSet` writes a plain include's engine blocks in place: `TestReadSet` row `PlainIncludeCarriesItsEngineBlocksInPlace`.
- A plain include that does not parse now fails in stencilstore: `TestTable_CheckMarkers` row "an included block that does not parse" (expected message updated).
- `termsIn`, for each term form and for the negatives (imperative Write/Read, a card's Edit, `spawn_agent`): `TestClaudeTermScans_SyntheticRows/TermsInNamesEachToolForm`.
- The stencil scan skips the Claude blocks and the `orch-` stencils: subtest `StencilScanSkipsTheClaudeBlocksAndTheOrch`.
- The spec scan skips a spec's header; the Go literal scan reads raw strings and comments, skips `claudeengine` and `internal/orchcli`, admits allowlisted literals and reports stale entries: subtest `SpecAndGoLiteralScans`.
- The Go scan skips `internal/testkit/` and `contracts/orchrole/`: none (no synthetic row).
- Missing engine-block copies, for registered blocks and for include-only blocks: subtest `EngineBlockWithoutACopy`.
- Shipped stencils, specs and literals comply: `TestStencils_ClaudeTermsOnlyInClaudePlaces`, `TestStencils_EveryEngineBlockHasACopyPerEngine` and `TestClaudeTermsOnlyInClaudePlaces_SpecsAndGoLiterals`.
- The gate fixer renders frame and body for `in.Engine`: `gatefixer/prompt_test.go:62` rows claude/codex. A wrong read would fail the render on the unresolved include.
- Weaver's batch template is read for `in.Engine`: `weaverengine/render_test.go:79` codex row and the claude row.
- Weaver's verify-fix body goes through `gatefixer` with the ladder engine: covered by the gatefixer rows above; `TestRun` covers the fake engine.
- Darn's Step 4 list is read for the writer's resolved engine: Claude only (`spec_test.go`, `ungatedSubagentGate`); no Codex row.
- The twill directive is read for `Values.Engine`: `TestDirective_RendersEveryValueVerbatim` covers Claude only; no Codex row and no empty-engine row.
- Loom passes `chairChoice.Engine` and `choice.Engine` to the twill directive: `TestPlanTable` with the default Claude engine only.
- Webster `BeginDeps.Engine` from Master's role, used by `RenderForkPrompt`: none that tells Master's engine from another. Fixtures use `"claude"` or the fake default engine, and `runlevel_test.go:1583` gives Master and recovery the same engine `a`.
- Webster's verify-fix prompt is rendered for Master's engine (moved up in `runlevel.go`): none that tells which engine is used, for the same reason.
- The new `webstercli` `beginDeps` refusal "no resolved model-spec for role master": none.
- `MerriamBase` reads the Master stencil raw: `TestMerriamBase` (`merriambase_test.go:94-110`).
- The reworded Go texts:
  - fork audit message: `TestAuditClusterRound` (`nested spawn call`) and `TestSmokeBurlerClusterRogueFork`
  - twill fixer refusal: `TestEngine_Fix_Outcomes` (`needs forks`)
  - verify-module-wide detail: `TestValidate_VerifyModuleWide`
  - `modeRules`, the drift operator mode and the help texts: no row, which is right since they are static prose.
- The chair seat's engine in the seat test fixture (`asAssistants`): test support only, so no row is needed.

**Findings**

1. MEDIUM — design — engine selection per producer (`internal/darnengine/spec.go:120`, `internal/loomengine/discussiontable.go:126`, `internal/loomengine/plan.go:119`, `internal/webstercli/beginbatch.go:40-45`, `internal/websterengine/runlevel.go:724-729`)
   - The diff adds a behaviour in each place: the prompt is worded for the engine the launch resolves to (darn writer, twill directive, Master for the begin-batch fork prompt and the verify-fix prompt).
   - Every test of these renders only the Claude or the fake default engine, or gives every role the same engine.
   - So no row fails if a producer passes the wrong engine, such as recovery's or the default instead of Master's.
   - Evidence: `darnengine/spec_test.go:61` (`DefaultEngine: "claude"`), `twilldirective_test.go:34` (`Engine: "claude"`), `beginbatch_helper_test.go:109` (`Engine: "claude"`), `webstercli/verbs_test.go:125-126` (Master role with no engine), `runlevel_test.go:1583-1584` (Master and recovery both `a`).
   - Gatefixer and weaver are the exception: their claude/codex rows (`gatefixer/prompt_test.go:62`, `weaverengine/render_test.go:79`) show the pattern.
   - Fix: add one Codex or two-engine row per producer that asserts the chosen engine's block copy, as `RequireEditDirectiveOf` does for the edit directive.
2. LOW — design — `internal/twilldirective/twilldirective.go:52-54` and `doc.go:50`. The documented refusal of an empty `Values.Engine` has no test row. `filledValues` always sets `"claude"` (`twilldirective_test.go:34`).
3. LOW — design — `contracts/stencils/engineterms_test.go:277-282`. The `skippedGoPrefixes` entries `internal/testkit/` and `contracts/orchrole/` have no synthetic row in `SpecAndGoLiteralScans` (lines 518-525 cover only `claudeengine` and `internal/orchcli`), so dropping either skip is unpinned.
4. LOW — design — `internal/webstercli/beginbatch.go:40-43`. The new refusal when Master's role is unresolved has no test row. `verbs_test.go:125-126` always sets the Master role.

Tool calls:
1. Grepped the producer tests (darn, gatefixer, twilldirective, weaver, loom, webster) for Codex and engine rows, which the manifest did not list.
2. Grepped `merriambase_test.go` and the `webstercli` begin-batch tests for the MerriamBase coverage and the Master role fixtures, which the manifest did not list.
