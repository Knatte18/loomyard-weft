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
The websterengine doc names the `card`, `batch`, `webster` and `webster-fix` keys and the `--gate` rendering where it names the `final` and `round` gates and `--tier <n>...`, and the gateslot doc says `GateCall` renders `--gate <key>` with one `--card <id>` per card.

`PATTERN.md`'s `PATTERN-test-tiers` line becomes this line, verbatim:

```
- `PATTERN-test-tiers` — Running hub's tests: only via `lyx gate test`; a site names its gate key, never tiers or scope, bar darn's tree-scoped targets. — [background](pattern/PATTERN-test-tiers.md)
```

Its background file says a site or stencil names a gate key and never a tier list, scope form, test command or target list, and that darn's tree-scoped sites add the touched trees' targets to the key's call.
`webster-spec.md` and the Webster master and implementer stencils name the `webster` gate where they name the `final` gate.

**ImpactSummary:** Every rendered gate call changes from a tier list plus scope words to one `--gate <key>` call; webster's card, batch and end gates, `lyx webster verify` and loom's gate sources move together.
