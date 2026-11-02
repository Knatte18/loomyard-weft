# Card 3 — webster-fixer-on-core

**Create:**
- `plan:internal/websterengine#verifyFixerStore` -> `type verifyFixerStore struct`
- `plan:internal/webstercli#websterCLI.pairSlug` -> `func (c *websterCLI) pairSlug() string`
- `plan:internal/webstercli#TestWebsterCLI_PairSlug` -> `func TestWebsterCLI_PairSlug(t *testing.T)`

**Delete:**
- `internal/websterengine/coldfixer.go`
- `internal/websterengine/strand_test.go`
- `internal/websterengine#StrandLive`
- `internal/websterengine#TurnEndRead`
- `internal/websterengine#TurnEndedAfter`
- `internal/websterengine#turnEndSettle`
- `internal/websterengine#verifyFixerStrandRole`
- `internal/websterengine#RenderVerifyFixColdPrompt`
- `internal/websterengine_test#renderVerifyFixCold`
- `contracts/stencils/webster/webster-prefix-verify-fix-cold.md`
- `contracts/stencils#WebsterPrefixVerifyFixCold`

**Edit:**
- `internal/websterengine#FixVerifyDeps`
- `internal/websterengine#FixVerifySpawnOrAttach`
- `internal/websterengine#FixVerifyAwait`
- `internal/websterengine#FixPlanAwait`
- `internal/websterengine#awaitTerminal`
- `internal/websterengine#refuseReportHeadBatch`
- `internal/websterengine#writerRecovery`
- `internal/websterengine#RecoverSpawnOrAttach`
- `internal/websterengine#ClassifyInputs`
- `internal/websterengine/doc.go`
- `internal/webstercli#websterCLI.fixVerifyDeps`
- `contracts/stencils/webster/webster-body-verify-fix.md`
- `contracts/stencils#entries`
- `contracts/stencils#WebsterBodyVerifyFix`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `internal/websterengine_test#TestFixVerifySpawnOrAttach`
- `internal/websterengine_test#TestRenderPrompts`
- `internal/websterengine_test#TestRenderPrompts_FrictionDirective`
- `internal/websterengine_test#TestRenderPrompts_ParentDirective`
- `internal/websterengine_test#testReplyPath`

**Uses:**
- `internal/websterengine#VerifyFixerState`
- `internal/websterengine#RecordVerifyFixerTerminal`
- `internal/websterengine#verifyFixValues`
- `internal/websterengine#composeVerifyFixTemplate`
- `internal/websterengine_test#TestFixVerifyAwait`
- `internal/websterengine_test#TestFixPlanAwait`
- `internal/webstercli/doc.go`
- `internal/gatecli#originSlug`
- `internal/hubforge#`

**Intent:**
Move `lym webster fix-verify` onto `internal/gatefixer` (D6).
Its verb, trigger, fork-first ladder, recovery-role model, `verify-fixer` role, `recovery_timeout_min` and `VerifyFixers` records stay; one behavior changes: its strand is registered with runproc under the worktree's pair slug.

- `verifyFixerStore` adapts `*State` to `gatefixer.Store`: `Live` returns the last `VerifyFixers` record converted to a `gatefixer.Record` when its `Status` is empty, and `Save` appends a record converted back to `*VerifyFixerState`, with report and reply paths dropped since webster derives both from its geometry.
- `FixVerifySpawnOrAttach` keeps its signature, its refusals and its order of checks, then renders the prompt with `gatefixer.RenderPrompt`, frame first and `webster-body-verify-fix` as the body, its body values from `verifyFixValues` and its friction note `webster-verify-fix-cold` as today, and spawns through `gatefixer.SpawnOrAttach` over a `verifyFixerStore` on `st`, with `Role: gatefixer.RoleVerifyFixer`, the recovery role's model choice, `roleSkills` plus the language skills, `segmentcolor.Webster` and `recovery_timeout_min`.
  It returns the record `st.VerifyFixers` now holds, and whether it spawned.
  An error from `gatefixer.SpawnOrAttach`, a start, resolve, registration or save failure, is returned with the way forward `transient, re-run `lym webster fix-verify``, as the verb's spawn errors carry today.
  The caller still saves `st`; since the store's `Save` only appends to the in-memory state, the CLI's save-then-stop-on-failure stays the strand's guard.
- `FixVerifyDeps` gains `Register func(guid string) error`, passed as `gatefixer.Deps.Register`.
- `FixVerifyAwait` and `FixPlanAwait` call `gatefixer.Await` over a `gatefixer.Record` built from their state record and reply path, with `recovery_timeout_min`, and copy its `Result` into their own result types; the plan fixer keeps its own role and state.
- `awaitTerminal` and `refuseReportHeadBatch` call `gatefixer.TurnEndedAfter`, `gatefixer.TurnEndRead` and `gatefixer.StrandLive` in place of the deleted webster copies; `strand_test.go`'s two tests moved to `gatefixer` in card 1.
- `coldfixer.go`, `RenderVerifyFixColdPrompt`, its test helper `renderVerifyFixCold`, `verifyFixerStrandRole` and the cold prefix stencil with its embed and registry entry are deleted.
  The prefix's two webster-only orientation lines join `webster-body-verify-fix.md`, where they bind the fork too: read `_lym/plan/00-overview.md` before editing, and never run a `lym webster` command; the body's leading comment names the shared frame in place of the cold prefix.
- `roleOpeningStencils` maps `verify-fixer` to `gatefix-frame-cold`; `TestStencils_TopLevelMarkers` drops the cold prefix's row.
- `websterCLI.pairSlug()` returns the worktree's pair slug when the geometry is in a hub and `fabricengine.ReadOrigin` finds the worktree's fabric origin, as `originSlug` reads it, and `""` otherwise; card 14 reads it too.
- `websterCLI.fixVerifyDeps` sets `Register` to `gatefixer.Registrar(c.reed, fabricengine.RunProcDir(hub), c.pairSlug(), gatefixer.RoleVerifyFixer)`, which is nil for an empty slug (card 1), so a standalone or prime run is unregistered as today.
- `writerRecovery` and the contract-file refusal in `RecoverSpawnOrAttach` name the verify fixer's role through `gatefixer.RoleVerifyFixer` in place of the literal `verify-fixer`, so the moved constant is the one spelling.
- `doc.go` names `internal/gatefixer` as the owner of the cold fixer's spawn, wait and registration, and the shared frame in place of `RenderVerifyFixColdPrompt`'s prefix.
- These comments drop the deleted names, naming `gatefixer`'s members or the shared frame in their place; the file headers of `render.go`, `strand.go` and `recoverbatch.go` are card 4's:
  - the `testReplyPath` doc comment, which names `RenderVerifyFixColdPrompt`'s `replyPath` parameter and becomes the placeholder reply path the cold fixer prompts are rendered with, the verify fixer's through `gatefixer.RenderPrompt` and the plan fixer's through `RenderPlanFixPrompt`;
  - `doc.go` lines 423 and 560, which name `TurnEndedAfter` and `RenderVerifyFixColdPrompt`;
  - the `ClassifyInputs.TurnEnded` field comment, which names `TurnEndedAfter`, moved above the field;
  - the `WebsterBodyVerifyFix` doc comment, which names `WebsterPrefixVerifyFixCold`.

Tests, tier0 as today:
- `TestFixVerifySpawnOrAttach` gains rows: a spawn calls `Register` with the strand's guid; a `Register` failure stops the strand and returns the error, naming the transient re-run way forward, with nothing appended to `VerifyFixers`; the started spec's role is `gatefixer.RoleVerifyFixer`; the rendered prompt carries the frame's reply-file line and the body's report path.
- `TestWebsterCLI_PairSlug`, tier1, over a `hubforge` hub: a pair worktree's CLI answers its slug, and the prime's answers `""`.
- `TestRenderPrompts`, `TestRenderPrompts_FrictionDirective` and `TestRenderPrompts_ParentDirective` render the cold fixer through `gatefixer.RenderPrompt` with the webster body in place of `renderVerifyFixCold`: the empty-reply refusal row, the friction row and the parent-directive row keep their assertions.
- `TestFixVerifyAwait` and `TestFixPlanAwait` stand unchanged: the classification and stop behavior they pin is the same.

**ImpactSummary:** webster's cold fixer and plan fixer wait through `gatefixer`; every caller of the deleted members is in `internal/websterengine` and listed here; the one behavior change is the runproc registration of the verify fixer on a pair.
