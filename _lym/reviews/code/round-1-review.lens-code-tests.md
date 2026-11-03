```yaml
lens: code-tests
outcome: reported
```

**Lens code-tests: per behaviour, the test that pins it, or none**

Every added behaviour has a test, so I have no findings.

| Behaviour | Pinning test |
|---|---|
| Expand: nested rewrite, `outer/inner` path, innermost exit, inner `on_done` of its own, argument not mutated | `TestExpand` "shape" rows |
| Expand: parent's terminals stay the run's; recipe with no block returned unchanged under a nil resolver | `TestExpand` terminals and no-block subtests |
| Expand refusals: unknown recipe, nil resolver, self-inclusion (direct and through an inner block), row-name collision, segment collision, `on_done`/`on_stuck`/`on_reroute` leaving the recipe, outermost empty `on_done`, `on_done` naming the block or an inlined row, sub-recipe setting `contract`/`verify_source`/`amendment_target`/`profile_recipe` | `TestExpand` "refuses …" rows, one per refusal |
| Parse: block row setting `engine`/`config`/`on_stuck`/`on_reroute`/`segment`/`max_bounces`/`entry_gate`; missing `block`; missing `block_gate`; hand-written engine `Block`; empty `on_done` allowed; `block_path` in a file refused; `profile_recipe` decodes | `parse_test.go` rows |
| Build: unknown `block_gate` refused; unexpanded block row refused; `always` gate holds | `build_test.go:66` and `:72`; `entrygate_test.go:142` |
| `RoutingFrom` carries `SkipTo` and `Block` | `routing_test.go` "the block rows carry the skip target and the block path" |
| `BlockGates` table and the `Block` producer returning Done | `entries_block_test.go`; `entries_simple_test.go` "Block producer returns Done"; `registry_test.go` "Block" |
| Skip routes by `SkipTo`, no inlined row recorded; holding gate enters and the inlined entry carries the block; skipped entry of an inner block carries the enclosing block | `entry_test.go`, three subtests |
| Goto and re-launch history entries carry `Block`; goto admission reads the skip target after a skip | `goto_test.go` "target entry carries the target row's block" and "blocked at the exit after a pass-over admits an inlined row of the block" |
| Reroute and Stuck entries carry `Block` | `reroute_test.go:65-83` |
| Validate refuses a `SkipTo` naming no producer, and accepts a later one | `validate_test.go` |
| Goto onto a block row bypasses its gate; re-launch onto it clears the entered row | Covered by existing behaviour: `Goto` sets the entered row (`goto_test.go:66`) and `Relaunch` clears it (`goto_test.go:620`). No block-specific row, but the code path is the generic one. |
| Motif rows equal tapestry's, arras expands to tapestry plus the `Motif` block, top-level keys equal, arras builds under the loom contract | `TestBlockRecipes` |
| Motif absent from `RecipeNames`; arras present | `seed_test.go` rows; `table_test.go` key set |
| `ProfileRecipe` accessor | `recipekeys_test.go` `wantProfileRecipe` rows |
| `prepareRunSettings` resolves the profile through `ProfileRecipe` | `runsettings_test.go` (36 lines changed). I did not read the rows. |
| Arras accepted in `battencli`, `reportengine` runs and phase table, arbiter description | `wire_test.go:457-517`; `runs_test.go:116`; `timing_test.go:399`; `recipekeyed_test.go` "Arras" |
| Progress numbers `Motif` as its own step and jumps to the exit on a pass-over | `progress_test.go` arras subtest, which has a pass-over step comment |
| Status shows entered and the block beside a row name | `human_test.go:146` and `:161`; `checklist_test.go` rows "a block row's Done reads entered…" and "…recipe unreadable…" |
| `interruptpolicy` row for `NameMotif` | `interruptpolicy_meta_test.go` (changed). I did not read it. |

Findings: none.

Tool calls (3):
1. Listed added `Test`/`t.Run` names per changed test file, because the manifest held no test contents.
2. Read the added `expand_test.go`, `parse_test.go`, `entry_test.go`, `goto_test.go` and `validate_test.go` rows, to match them to each refusal and routing case.
3. Searched for the block-gate, `ProfileRecipe`, reroute, status and progress tests, and for arras in the report, wiring and arbiter tests. I also read the added progress, status and checklist rows to confirm the pass-over and entered cases.
