# Card 1 — gate-key-shape

**Edit:**
- `internal/gateslot#Config`
- `internal/gateslot#configFile`
- `internal/gateslot#decodeConfig`
- `internal/gateslot#MigrateConfig`
- `internal/gateslot#goReadGates`
- `internal/gateslot#Config.TierList`
- `internal/gateslot#Config.TiersValue`
- `internal/gateslot/template.yaml`
- `internal/gateslot#TestLoadConfig`
- `internal/gateslot#TestMigrateConfig`
- `internal/gateslot#TestConfig_TierUse`
- `internal/gateslot#TestTemplateConfigAndLimits`
- `internal/websterengine#NewVerifyGate`
- `internal/websterengine#FixVerifySpawnOrAttach`
- `internal/websterengine#Run`
- `internal/webstercli#websterCLI.runVerify`
- `internal/loomcli#planVerifySource`
- `internal/loomcli#darnVerifySource`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/landingshed#verifyGate.recordFailure`
- `internal/gateslot/doc.go`
- `internal/websterengine#gateTestConfig`
- `internal/websterengine_test#testGateConfig`
- `internal/websterengine_test#TestFixVerifySpawnOrAttach`
- `internal/websterengine_test#gateScenario.restart`
- `internal/websterengine#verifyGateConfig`
- `internal/webstercli#verifyGateConfig`
- `internal/webstercli#TestGateParity_VerifyGate`
- `internal/configcli#TestDispatchLayers`
- `internal/configcli#TestConfigOverRealHub`
- `internal/loomcli#TestGateListSource`
- `internal/loomcli#gateConfigWithTwoTiers`
- `internal/loomcli#TestLandingDeps_VerifyCommandReadsPlanAtCallTime`
- `internal/landingshed#fixedGateTierLists`
- `internal/landingshed#TestMergeIn_PlanVerifyFailureRecord`
- `internal/loomshed#roundGateConfig`
- `internal/gateslot#templateDefaults`
- `internal/websterengine#TestCardGateCommand`
- `internal/websterengine_test#TestRecoverSpawnOrAttach`
- `internal/websterengine_test#TestBeginBatch_PromptFile`
- `internal/loomshed#roundGateFixture.round`

**Create:**
- `plan:internal/gateslot#GateKey` -> `type GateKey struct`
- `plan:internal/gateslot#Config.Gate` -> `func (c Config) Gate(key string) (GateKey, error)`

**Intent:**
Reshape `gate.yaml`'s `gates` (D1, D2) from a name-to-list map into one key per producer, each `{tiers: [<n>...], scope: card|diff|all}`.

`GateKey` holds `Tiers []int` and `Scope string` with the yaml tags `tiers` and `scope`; `Config.Gates` becomes `map[string]GateKey`, and `configFile` decodes the same shape.
`gates` stays a per-name map, so a hub file that sets one key keeps the template's value for every other key.
`decodeConfig` refuses, naming the key and `lyx config gate`, a scope that is not `card`, `diff` or `all`, and a `card` scope on any key but `card` and `batch`: "gate config key gates.<key>: scope card is only for the card and batch keys; fix it with lyx config gate from the prime" plus the role clause.
The input that reaches it is a hub file holding `merge-in: {tiers: [0], scope: card}`; a `TestLoadConfig` row reaches it, and another row reaches the unknown-scope refusal with `scope: some`.
A `gates` entry still in the retired list shape, such as `card: [0]`, which a hub file holds after a deploy until reconcile migrates it, is refused naming the key and the way forward of `lyx fabric reconcile` (or `lyx config reconcile --layer default --apply`) from the prime, which runs `MigrateConfig`, never `lyx config gate`, plus the role clause; a `TestLoadConfig` row loads a hub file holding `card: [0]` and asserts the reconcile way forward.

`Config.Gate` returns a key's tiers and scope with the refusals `TierList` gives today (no `tier0`, no such key, an empty tier list, an undeclared tier), and `TierList` returns `Gate(key).Tiers`, so every site that reads a list keeps working.
`GateCall` keeps rendering `--tier <n>...` plus the given scope words in this card; card 3 switches it to `--gate`.

The template's `gates` holds every key the shipped recipes read, each `{tiers: [0]}`: `card` and `batch` with scope `card`, `webster`, `merge-in` and `finalize` with scope `all`, and `webster-fix`, `darn-write` and `code-burler-fix` with scope `diff`; its comment names the keys and scopes.
`goReadGates` lists those eight keys, and `TiersValue` renders each as `- gate <key>: [<n>, ...] <scope>`.

`MigrateConfig` rewrites a hub file's old lists through yaml node editing, keeping comments and every other key: `landing`'s list becomes `merge-in` and `finalize` with scope `all` and `darn-write` and `code-burler-fix` with scope `diff`; `final`'s becomes `webster` with scope `all`; `round`'s becomes `webster-fix` with scope `diff`; a list under `card` or `batch` becomes `{tiers: <list>, scope: card}`; `gates.publish` is still pruned.
A key already in the mapping shape is left alone, so a second pass changes nothing; each rewrite returns one line, such as `gates.landing: retired, migrated to merge-in, finalize, darn-write and code-burler-fix`.
A migrated key that the file already holds in the new shape keeps its value.
`TestMigrateConfig` gains rows for a file holding `landing`, `final`, `round`, `card` and `batch` lists, asserting the keys and scopes D1 names, for a second pass that changes nothing, and for a file holding both `landing: [0, 1]` and `merge-in: {tiers: [0], scope: diff}`, asserting `merge-in` keeps its value.

The sites keep their behavior under the new key names, so a template-only config still renders a call: Webster's end gate and `lyx webster verify` read `webster` where they read `final`; the cold fixer (`FixVerifySpawnOrAttach`) and the fixer fork (`Run`) read `webster-fix` where they read `round`.
Until cards 4 and 5 give each row its own key, loom's and darn's landing sources read `finalize` where they read `landing`, loom's base command reads `webster` where it read `final`, the fix rows' `Round` source reads `code-burler-fix`, and `verifyGate.recordFailure` looks up the tiers of `finalize` for a log that names no failing tier.
The test fixtures listed above, found by a search for `Gates`, `TierList(` and `gates:` seeds over the module's test files, move onto the new shape and names in this card, so every package compiles and loads: gateslot's `templateDefaults` builds `Gates` as `map[string]GateKey` holding the template's eight keys, the Webster helpers `gateTestConfig`, `testGateConfig` and `TestFixVerifySpawnOrAttach` build it as `map[string]gateslot.GateKey`, and `TestCardGateCommand`, `TestRecoverSpawnOrAttach` and `TestBeginBatch_PromptFile` set their dangling `card` and `batch` entries as a `gateslot.GateKey` with `Tiers: []int{7}`; `roundGateFixture.round` reads `finalize` where it reads `landing`, until card 4 deletes it; the seeds `gateScenario.restart`, both packages' `verifyGateConfig`, `TestConfigOverRealHub`, `gateConfigWithTwoTiers` and `roundGateConfig` write keyed `gates` entries; `TestDispatchLayers` compares `cfg.Gates["finalize"].Tiers`; `TestGateParity_VerifyGate` renders `GateCall("webster", "--all")`; `TestGateListSource` reads `code-burler-fix`, and `TestLandingDeps_VerifyCommandReadsPlanAtCallTime` rewrites and asserts the `finalize` key; `fixedGateTierLists` answers `finalize` for `TestMergeIn_PlanVerifyFailureRecord`'s tierless log.
The gateslot doc's Config section describes the keyed `gates`, the scopes, the `card` scope's restriction, the list-shape refusal and the migration in this card.

**ImpactSummary:** Changes the shape of `gate.yaml`'s `gates` and the template, read by every gate site; sites keep their calls under the new key names.
