# Card 9 — webster-declared-changes

**Edit:**
- `internal/websterengine#Geometry`
- `internal/hubgeom#WebsterGeometry`
- `internal/websterengine#RenderRecoveryPrompt` -> `func RenderRecoveryPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, failureDigest, uncommittedPaths, reportPath, repoRoot, planDir, promptWorktreeRoot, stencilsDir, specsDir string, verifyTiers, patternChanges []string, selfFixCap int, notePath, parentName string) ([]byte, error)`
- `internal/websterengine#RenderMasterPrompt` -> `func RenderMasterPrompt(batches []batcher.Batch, st *State, outcomePath, summaryPath, verifyFixPromptPath, planDir string, selfFixCap int, worktreeRoot, repoRoot, stencilsDir string, patternChanges []string, notePath, parentName string) ([]byte, error)`
- `internal/websterengine#recoverSpawn`
- `internal/websterengine#Run`
- `internal/websterengine#MerriamBase`
- `internal/websterengine/doc.go`
- `internal/websterengine_test#renderRecoveryWithUncommitted` -> `func renderRecoveryWithUncommitted(batch batcher.Batch, repoRoot, anchorRoot, stencilsDir, specsDir, failureDigest, uncommittedPaths, notePath, parent string, verifyTiers, patternChanges []string) (string, error)`
- `internal/websterengine_test#renderRecovery`
- `internal/websterengine_test#renderMaster` -> `func renderMaster(batches []batcher.Batch, anchorRoot, repoRoot, stencilsDir, notePath, parent string, patternChanges []string) (string, error)`
- `internal/websterengine_test#TestRenderPrompts`
- `internal/websterengine_test#TestRenderPrompts_FrictionDirective`
- `internal/websterengine_test#TestRenderPrompts_ParentDirective`
- `internal/websterengine_test#TestRecoverSpawnOrAttach`
- `internal/websterengine_test#TestRun_RefusesBeforeSpawn`
- `internal/websterengine_test#TestMerriamBase`
- `internal/hubgeom#TestWebsterGeometry`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `plan:internal/discussionparser#DeclaredPatternChanges`
- `plan:internal/discussionparser#ErrPatternChangesInvalid`
- `internal/websterengine_test#patternActiveLayout`
- `internal/websterengine_test#newRecoverFixture`

**Intent:**
D5.4 for webster: the Master's orchestrator directive and the recovery strand's implementer directive render the task's declared PATTERN changes, read from the decision record at a path the geometry is told (`PATTERN-told-geometry`).
The in-session forks inherit the Master's directive, so they see the same changes.

- `Geometry` gains `DecisionRecordPath`, the told decision record, empty for none.
  `hubgeom.WebsterGeometry` fills it with `loomengine.DiscussionDecisionRecord(l)`, the one declarer of that path; `standalonegeom` leaves it empty, so a standalone webster run renders no changes, as D5.4 says.
- `RenderRecoveryPrompt` and `RenderMasterPrompt` gain `patternChanges`, passed to `pattern.Directive` as its `declared` argument in place of card 8's nil.
- `recoverSpawn` calls `discussionparser.DeclaredPatternChanges(deps.Geom.DecisionRecordPath)` before it reads HEAD or mutates anything; an error refuses the spawn as `<err> re-run `lyx webster recover-batch NN``, completing the way forward the error carries, with nothing spawned or recorded.
  The input that reaches it: a record whose `## PATTERN changes` holds a bullet with an unknown action, past every check `recover-batch` makes before the spawn.
- `Run` reads it the same way just before `RenderMasterPrompt`, and an error ends the run before Master's spawn with the way forward completed by `deps.reentryStep()`.
- `MerriamBase` reads it too and passes the result to its `pattern.Directive` call, so the start base sizes the directive the Master gets; a record that fails to parse leaves the changes out of the base, since the master prompt render refuses on it and `MerriamBase`'s callers name a transient way forward that does not fit a record fix.
- `webster-template-master.md`'s orientation sentence "they stay current for the whole run because the run refuses a drifted plan" narrows to the plan the drift check covers, and adds that the declared PATTERN changes rendered in the constraints section above cover PATTERN.
- `refusal-spec.md` `## webster` gains a row: a `recover-batch` spawn or a `run` finds the decision record's `## PATTERN changes` malformed; correctness halt; way forward: the operator fixes and commits the section (the orch escalates the refusal to the operator), confirms it with `lyx loom validate-discussion`, then re-runs the verb.
- `doc.go` says the master and recovery prompts render the declared changes from `Geometry.DecisionRecordPath`, empty in standalone mode.

Tests:
`renderRecoveryWithUncommitted` and `renderMaster` gain a `patternChanges` argument; `renderRecovery` passes nil through, and the direct and helper calls in `TestRenderPrompts`, `TestRenderPrompts_FrictionDirective` and `TestRenderPrompts_ParentDirective` pass nil, so those tests' existing rows pin the no-declaration render.
`TestRenderPrompts` gains rows over `patternActiveLayout`: a recovery and a master prompt rendered with one declared bullet carry it under the declared-changes heading.
`TestRecoverSpawnOrAttach`, beside its existing refusal rows (the uncheckable-findings refusal), gains a row whose setup writes a record with an unknown-action bullet under the fixture's worktree and points `fx.Deps.Geom.DecisionRecordPath` at it: the refusal wraps `ErrPatternChangesInvalid`, names `lyx loom validate-discussion` and `lyx webster recover-batch`, the prior record is unchanged and `fx.Engine.PrepareCalls` stays zero.
`TestRun_RefusesBeforeSpawn` gains the same record as a row: the run returns the refusal and Master is never spawned.
`TestMerriamBase` gains a row whose record declares one bullet, where the base's line count grows by the rendered fragment's lines over the same geometry with no record, and a row whose record's section is malformed, where `MerriamBase` returns no error and the base equals the no-record base.
`TestWebsterGeometry` asserts `DecisionRecordPath` equals `loomengine.DiscussionDecisionRecord` of the location.

**ImpactSummary:** two re-signed renderers, each with one production caller (`recoverSpawn`, `Run`) and one test helper; a hub run now reads one more file before each recovery spawn and the master spawn.
