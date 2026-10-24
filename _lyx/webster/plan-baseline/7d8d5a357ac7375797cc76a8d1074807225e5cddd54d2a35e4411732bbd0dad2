# Card 4 — verify-gate-site-key

**Edit:**
- `internal/loomshed#NewVerifyGate` -> `func NewVerifyGate(worktreeRoot, verifyDir, siteLabel string, command, mergeBase func() (string, error), treeScope func() (string, bool, error), hold *verifytree.PauseHold) shuttleengine.Gate`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolveRowGateSpec`
- `internal/shedrecipe#resolveGateSpec`
- `internal/shedrecipe#TestResolveGateSpec_Rejections`
- `internal/loomshed#roundCall`
- `internal/loomshed#roundGateConfig`
- `internal/loomshed/doc.go`
- `internal/shedrecipe#resolveGateEntry` -> `func resolveGateEntry(entry, row, gateKey string, index int, elem Config, env Env) (shuttleengine.GateEntry, error)`
- `internal/shedrecipe#resolvePlainGateClosure` -> `func resolvePlainGateClosure(entry, row, gateKey string, index int, name string, env Env) (shuttleengine.Gate, error)`
- `internal/shedrecipe#darnWriteEntry`
- `internal/shedrecipe#burlerFixEntry`
- `internal/loomcli#gateCallSource` -> `func gateCallSource(dirs configengine.Dirs) func(key string) (string, error)`
- `internal/loomcli#planVerifySource`
- `internal/loomcli#darnVerifySource`
- `internal/loomcli#darnTreeScope` -> `func darnTreeScope(l *lyxcwd.Location) func(gateKey string) (string, bool, error)`
- `internal/loomcli#TestDarnTreeScope_ClassifiesAtEachCall`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/testkit/envkit#FullEnv`
- `internal/loomshed#TestVerifyGate_Scenario`
- `internal/loomshed#TestVerifyGate_RoundScenario`
- `internal/shedrecipe#TestResolveGateSpec_ValidOneEntryPerVocabularyName`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/loomshed#roundGateFixture.runGate`
- `internal/loomshed#roundGateFixture.diffGate`
- `internal/webstercli#TestGateParity_VerifyGate`
- `internal/shedrecipe#newTestEnv`
- `internal/shedrecipe#TestResolveGateSpec_VerifyRequiresAbsoluteEnvRoots`
- `internal/battencli#intentionallyNil`
- `contracts/recipes/loom-recipe.yaml`
- `contracts/recipes/darn-recipe.yaml`
- `pattern/PATTERN-verified-tree.md`

**Delete:**
- `internal/loomshed#RoundGate`
- `internal/loomcli#gateListSource`
- `internal/loomcli#TestGateListSource`
- `internal/testkit/envkit#fixedRoundGate`
- `internal/shedrecipe#fixedRoundGate`
- `internal/loomshed#toldRoundGate`
- `internal/loomshed#roundGateFixture.round`

**Create:**
- `internal/loomcli#TestGateCallSource`

**Intent:**
The fix rows' and the darn writer's verify gate runs its own row's gate key (D1, D3).

A row whose `gates` list holds a `verify` entry reads the key its config's `gate_key` names, else its row name in lower case: `code-burler-fix` for Code-Burler-Fix and `darn-write` for Darn-Write.
`resolveRowGateSpec` computes it from `cfg["gate_key"]` (a string; another type is refused naming the row, the key and the way forward of setting `gate_key` to the name of a `gate.yaml` key, a string, or removing it to run the row name's key) and passes it through `resolveGateEntry` to `resolvePlainGateClosure`, whose `verify` case hands `loomshed.NewVerifyGate` the told command `func() (string, error) { return env.VerifyCommand(gateKey) }` and the tree scope `env.VerifyTreeScope(gateKey)` when that seam is set.
`burlerFixEntry` and `darnWriteEntry` accept `gate_key` among their keys, and `darnWriteEntry` calls `resolveRowGateSpec` with its row's name, so its site label reads `Darn-Write gate`.
Bound: `gate_key` selects only which key of the hub's `gate.yaml` the row runs; the key's tiers and scope stay hub-file only.

`Env.VerifyCommand` becomes `func(gateKey string) (string, error)` and `Env.VerifyTreeScope` becomes `func(gateKey string) (string, bool, error)`, each a recipe-wide lookup keyed by the row's key; `Env.Round` goes, with `loomshed.RoundGate` and `envkit`'s `fixedRoundGate`.
`envkit.FullEnv` fills the two keyed seams.
`TestWire_DarnRecipe` and `TestWire_DefaultConfig` drop their `Round().Render` assertions with the field, and battencli's `intentionallyNil` loses its `Round` row, so `TestWire_EverySeamFilled` lists no seam `envkit.NilSeams` no longer reports.

`loomshed.NewVerifyGate` loses its `round` argument.
In the round form the told command is the site's own `--gate <key>` call, run as it is, since the gate itself derives the `diff` base (card 8).
The comment lint keeps its own base (D3): the commit of the told command's latest recorded pass when it is an ancestor of HEAD, else the merge base `mergeBase` reads, so a first fix round lints rather than skips; with neither, no lint runs.
The darn form keeps its lint from the merge base on every attempt, runs the tree-scoped command the tree scope answers for a tree-scoped diff, and otherwise the told command.
`TestVerifyGate_RoundScenario` is rewritten onto the told command running as it is: `roundGateConfig` holds a keyed `code-burler-fix` entry, the told command is `roundCall()`, now the site's own `lyx gate test --wait --gate code-burler-fix` call run through the `lyx` `lyxbin.PutFirstOnPath` puts first, and the rows that asserted the stand-in `echo plan` command never runs and that `--all` or `--base` entries are recorded now assert the told command's own recorded passes.
It gains rows for a first round linting from the merge base, which fails on a fixed-column comment the round added, and a later round linting from its own recorded pass, which ignores a wrapped comment older than that pass; both scenario tests lose the round source.
`TestResolveGateSpec_ValidOneEntryPerVocabularyName` asserts the `verify` gate of a row named `Code-Burler-Fix` reads `VerifyCommand("code-burler-fix")` and that `gate_key: custom` makes it read `VerifyCommand("custom")`; `TestResolveGateSpec_Rejections` gains a row with `gate_key: 3`, refused naming the row, the key and the way forward.

`gateCallSource` returns a reader of a key's call, `GateCall(key)`, loading the config at each call, and replaces `gateListSource`: `loomCLI.wire` and `loomCLI.wireDarn` set `env.VerifyCommand` to it, and `planVerifySource` and `darnVerifySource` read their keys through it as in card 1.
`darnTreeScope` takes the key and, until card 6, ignores it and answers today's tree-scoped command; `TestDarnTreeScope_ClassifiesAtEachCall` calls the returned reader with a key.
`TestGateCallSource` replaces `TestGateListSource`: it pins that a key's call is read at each call, so an edit between two calls is seen, and that an unknown key returns the config refusal.

`resolveGateSpec`'s doc comment names the keyed `env.VerifyCommand` and `env.VerifyTreeScope` in place of `env.Round`.
The recipes' Code-Burler-Fix and Darn-Write comments, the loomshed package doc and `PATTERN-verified-tree.md` say each fix row and writer runs its own key, lints from its own latest pass or the merge base, and no longer shares Webster's end gate pass.

**ImpactSummary:** Re-signs the loom verify gate and two gate resolvers, keys two Env seams by gate key and drops the round source; touches shedrecipe, loomshed, loomcli wiring, envkit and battencli's seam list.
