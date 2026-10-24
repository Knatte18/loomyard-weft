# Card 3 — gate-call-renders-key

**Edit:**
- `internal/gateslot#Config.GateCall`
- `internal/gateslot#TestConfig_TierUse`
- `internal/websterengine#cardGateCommand`
- `internal/websterengine#renderBatchGate`
- `internal/websterengine#NewVerifyGate`
- `internal/webstercli#websterCLI.runVerify`
- `internal/loomcli#gateListSource`
- `internal/loomcli#gateCallSource`
- `internal/websterengine#TestCardGateCommand`
- `internal/websterengine#TestRenderBatchGate`
- `internal/loomcli#TestGateListSource`
- `internal/loomcli#TestLandingDeps_VerifyCommandReadsPlanAtCallTime`
- `internal/webstercli#TestGateParity_VerifyGate`
- `internal/websterengine_test#TestFixVerifySpawnOrAttach`
- `internal/websterengine#TestRenderCardGates_OneBulletPerCardWithAStepSubBulletEach`
- `internal/websterengine#TestRenameBoundary_BatchesCheckAndGateTheirOwnTree`
- `internal/websterengine_test#TestBeginBatch_PromptFile`
- `internal/websterengine_test#TestRecoverSpawnOrAttach`
- `internal/webstercli#TestRebaselineCmd_EditedCardOfFailedBatchThenRecover`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/loomcli#wantDarnLandingCall`
- `internal/loomcli#TestDarnTreeScope_ClassifiesAtEachCall`
- `internal/loomshed#roundGateFixture.round`
- `contracts/specs/loom-status-spec.md`
- `internal/websterengine/doc.go`
- `internal/gateslot/doc.go`
- `PATTERN.md`
- `pattern/PATTERN-test-tiers.md`
- `contracts/specs/webster-spec.md`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/webster/webster-body-implementer.md`

**Intent:**
Every Go-rendered site names its gate key, never tiers or scope (D3).

`Config.GateCall(key, cards...)` renders `lyx gate test --wait --gate <key>` followed by one `--card <id>` per card given, after checking the key through `Config.Gate`; the variadic words are now card ids, not scope words.
`cardGateCommand` passes its card's id and `renderBatchGate` every card's id, so the `card` and `batch` sites render `--card <id>` per card as D3 says.
Webster's end gate (`NewVerifyGate`'s verify command) and `lyx webster verify` render `GateCall("webster")` with no `--all`, since the key's `all` scope says it.
`gateCallSource` renders `GateCall(gate)` with no `--all`, and `gateListSource`'s `Render` ignores the scope words it is given and renders `GateCall(gate)` until card 4 removes it, so the fix rows' gate already runs the key's own scope.
`TestConfig_TierUse`, `TestCardGateCommand`, `TestRenderBatchGate`, `TestGateListSource` and `TestLandingDeps_VerifyCommandReadsPlanAtCallTime` assert the `--gate` spellings.
`TestGateParity_VerifyGate` asserts Webster's end gate and `lyx webster verify` render the one call `lyx gate test --wait --gate webster`, and `TestFixVerifySpawnOrAttach` asserts the cold fixer's prompt carries `lyx gate test --wait --gate webster-fix` in its `fix_gate` marker, the call the fixer fork renders through the same `GateCall`.
The other readers of the rendered call, found by a search of the module's tests and docs for `GateCall(` and for the old `gate test --wait --tier` literal, assert the `--gate` spelling in this card:
`TestRenderCardGates_OneBulletPerCardWithAStepSubBulletEach`, `TestBeginBatch_PromptFile`, `TestRecoverSpawnOrAttach` and `TestRebaselineCmd_EditedCardOfFailedBatchThenRecover` expect `lyx gate test --wait --gate card --card <id>` per card, and `TestRenameBoundary_BatchesCheckAndGateTheirOwnTree`, `TestBeginBatch_PromptFile` and `TestRecoverSpawnOrAttach` expect `lyx gate test --wait --gate batch` with one `--card <id>` per card of the batch;
`wantDarnLandingCall` becomes `lyx gate test --wait --gate finalize`, the landing call `TestWire_DarnRecipe` asserts through `VerifyCommand`, the `Round().Render` assertions of `TestWire_DarnRecipe` and `TestWire_DefaultConfig` expect `lyx gate test --wait --gate code-burler-fix`, the key card 1 gives the fix rows' round source, and `TestDarnTreeScope_ClassifiesAtEachCall`'s `landingCall` becomes `lyx gate test --wait --gate finalize`.
`roundGateFixture.round`'s `Render` ignores the scope words it is given and renders `GateCall` of its key, as `gateListSource`'s does, until card 4 deletes it, so no scope word is read as a card id.
`loom-status-spec.md`'s example of a running verify reads `Code-Burler-Fix: verify running 6m (attempt 2; lyx gate test --wait --gate code-burler-fix)`.
The websterengine doc names the `card`, `batch`, `webster` and `webster-fix` keys and the `--gate` rendering where it names the `final` and `round` gates and `--tier <n>...`, and the gateslot doc says `GateCall` renders `--gate <key>` with one `--card <id>` per card.

`PATTERN.md`'s `PATTERN-test-tiers` line becomes this line, verbatim:

```
- `PATTERN-test-tiers` — Running hub's tests: only via `lyx gate test`; a site names its gate key, never tiers or scope, bar darn's tree-scoped targets. — [background](pattern/PATTERN-test-tiers.md)
```

`PATTERN.md` already sits above its headroom line, so this card's `lint-pattern` step passes only when the overview is no longer than at HEAD.
In the same edit, these two entries take the shortened wording the parent's `changes` decisions of 2026-10-24 record, verbatim, which saves at least the line's growth; no rule changes:

```
- `PATTERN-trimpath-safe-root` — Locating module root in gate-built code: never from a `runtime.Caller` path alone.
- `PATTERN-role-skills-typed` — Loading spawned session's skill: spawner names it on launch spec and lyx delivers it; no stencil asks. (test)
```

Its background file says a site or stencil names a gate key and never a tier list, scope form, test command or target list, and that darn's tree-scoped sites add the touched trees' targets to the key's call.
`webster-spec.md` and the Webster master and implementer stencils name the `webster` gate where they name the `final` gate.

**ImpactSummary:** Every rendered gate call changes from a tier list plus scope words to one `--gate <key>` call; webster's card, batch and end gates, `lyx webster verify` and loom's gate sources move together.
