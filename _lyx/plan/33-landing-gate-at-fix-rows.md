# Card 33 — landing-gate-at-fix-rows

**Create:**
- `internal/loomcli#gateListSource`
- `internal/loomcli#TestGateListSource`

**Delete:**
- `internal/loomcli#roundGateSource`

**Edit:**
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomshed#RoundGate`
- `internal/loomshed#NewVerifyGate`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolvePlainGateClosure`
- `contracts/recipes/loom-recipe.yaml`
- `contracts/recipes/darn-recipe.yaml`
- `internal/loomshed#TestVerifyGate_RoundScenario`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_DefaultConfig`

**Extends:**
- `internal/loomshed/doc.go`

**Intent:** Every gate after Webster is the `landing` gate (D8): Darn-Write, Darn-Burler-Fix and Code-Burler-Fix run the `landing` call through `verifytree.Verify` where they ran the `round` call.
These rows' gate is the shared `verify` gate entry, whose told list source is `Env.Round`; `roundGateSource` becomes `gateListSource(dirs, gate)`, and `wire` and `wireDarn` tell it the `landing` gate, so the gate's narrowed call is the `landing` list over `--base`.
In the round form a diff with no usable base runs the list's own whole-tree call, `Render("--all")`, in place of the told command, which on loom is the `final` call; the told command stays the site's base command, the record entry a pass keeps, so the gate never runs a list other than `landing`.
At the darn rows a diff wholly under `tree_scope` keeps its tree-scoped command, as `NewVerifyGate`'s darn form already chooses it before the gate's call.
The `round` gate kind stays, since Webster's verify-gate fixer fork runs its call through `internal/websterengine`, which this card does not touch.
`loomshed.RoundGate`'s and `NewVerifyGate`'s docs and `Env.Round`'s say the told gate list is the `landing` list at these rows; the recipes' header comments and the verify-gate comments on those rows say the same.

`TestGateListSource` pins the source for the `landing` and `round` lists over a gate config, and a config refusal returned with its way forward.
`TestVerifyGate_RoundScenario` gains a row with no recorded pass, where the gate runs the told list's `--all` call and not the told command, over a gate config whose `landing` and `final` lists differ.
`TestWire_DarnRecipe` and `TestWire_DefaultConfig` each assert that `Env.Round` renders the `landing` list, so the fix rows of both recipes run the `landing` call.

**ImpactSummary:** One told source is renamed and repointed from the `round` list to the `landing` list; its two callers are the loom and darn wiring, and Webster's own gate is untouched.
