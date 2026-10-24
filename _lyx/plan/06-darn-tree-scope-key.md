# Card 6 — darn-tree-scope-key

**Edit:**
- `internal/darnengine#Config`
- `internal/darnengine#ConfigSpec`
- `internal/darnengine#normalizeTreeScope`
- `internal/darnengine#LoadConfig`
- `internal/darnengine/template.yaml`
- `internal/configreg#Modules`
- `internal/configcli#hubFileLoadRefusal`
- `internal/loomcli#darnTreeScope`
- `internal/loomcli#darnVerifySource`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/darnengine#TestLoadConfig`
- `internal/darnengine#TestTreeScopedCommand`
- `internal/darnengine#withTreeScope` -> `func withTreeScope(trees string) string`
- `docs/overview.md`
- `internal/configcli#TestDispatchHubWideBoard`
- `internal/configcli#TestDispatchLayers`
- `internal/configcli#darnWithTreeScope`
- `internal/loomcli#TestDarnTreeScope_ClassifiesAtEachCall`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/darnengine/doc.go`
- `internal/configcli/doc.go`
- `pattern/PATTERN-verified-tree.md`
- `pattern/PATTERN-test-speed.md`

**Delete:**
- `internal/darnengine#ErrTreeScopeWithoutTrees`
- `internal/darnengine#isSingleWaitingGateCall`
- `internal/darnengine#TreeScopedCommand`
- `internal/darnengine#treeScopeAddTreeCommand`
- `internal/darnengine#treeScopeOffCommand`
- `internal/darnengine#treeScopeVerifyCommand`
- `internal/darnengine#treeScopeTargetsMarker`
- `internal/darnengine#treeScopeControlOperators`
- `internal/configcli#holdsTree`

**Create:**
- `plan:internal/darnengine#TreeScopedTargets` -> `func TreeScopedTargets(cfg Config, paths []string) ([]string, bool)`
- `plan:internal/darnengine#MigrateConfig` -> `func MigrateConfig(existing []byte) ([]byte, []string, error)`
- `plan:internal/darnengine#TestMigrateConfig` -> `func TestMigrateConfig(t *testing.T)`

**Intent:**
Retire `darn.yaml`'s `tree_scope_verify` (D14): darn's tree-scoped sites run their own row's gate key over the touched trees' targets.

`Config.TreeScopeVerify` and the `tree_scope_verify` template line go; `ConfigSpec` lists `tree_scope_verify` among its retired keys, so the load drops it and reconcile prunes it.
Tree scope is on exactly when `tree_scope` holds a tree, and removing every tree turns it off; `ErrTreeScopeWithoutTrees`, the single-call check and the `{targets}` check go from `normalizeTreeScope`, which keeps the tree-key checks, and with them go the constants only those checks read, found by a search for each over `internal/darnengine`: the `{targets}` marker, the shell control operators and the three refusal commands.
The test helper `withTreeScope` loses its `verify` parameter, its callers passing only the trees.
`configcli.hubFileLoadRefusal` loses its clear-`tree_scope_verify`-first branch and `holdsTree`, so emptying `tree_scope` in the editor is an ordinary save; the configcli rows that asserted that refusal now assert the save is written.

`MigrateConfig`, registered as darn's `Migrate` in `configreg.Modules`, fires only while the hub file still holds `tree_scope_verify`, editing yaml nodes:
held empty, it empties `tree_scope` and prunes the key, keeping an off hub off;
held set, it prunes the key and keeps `tree_scope`;
absent, it changes nothing, so a later reconcile never touches trees the operator adds, and a hub whose file never wrote the key but holds trees turns tree scope on.
Each rewrite returns one line; `TestMigrateConfig` pins the three cases and that a second pass over each changes nothing, and a malformed document returns an error marked `configengine.ErrInvalid`.

`TreeScopedTargets` replaces `TreeScopedCommand`: for a diff wholly under the trees it returns the touched trees' non-empty targets, deduplicated, in tree-path order, and true; otherwise false.
`darnTreeScope` takes the row's key and renders `lyx gate test --wait --gate <key> <targets>` from the key's `GateCall` and the targets, and answers an empty command, which runs nothing, when no touched tree contributes a target.
Darn-Write, darn's Code-Burler-Fix and darn's Finalize thereby run their own key (`darn-write`, `code-burler-fix`, `finalize`) with the trees' targets in place of the key's scope.
Bound: a tree-scoped run's tiers are the row key's, not an operator-written command's.
`TestTreeScopedCommand` asserts the targets, `TestDarnTreeScope_ClassifiesAtEachCall` the rendered `--gate <key> <targets>` call per row key and, for a docs-only diff under a tree with an empty target, a scoped answer with an empty command that runs nothing, and `TestWire_DarnRecipe` the wiring.

The darnengine and configcli docs, `PATTERN-verified-tree.md` and `PATTERN-test-speed.md`'s tree-scope bound say a tree-scoped site runs its row's key over the trees' targets, and `docs/overview.md`'s loom entry, which names `tree_scope` and `tree_scope_verify` as scoping a non-production diff, says the darn gates run the row key over the touched trees' targets.

**ImpactSummary:** Removes `tree_scope_verify`, its refusals and the editor's clear-first branch, adds darn's config migration and keys the tree-scoped call by row; touches darnengine, configreg, configcli and darn wiring.
