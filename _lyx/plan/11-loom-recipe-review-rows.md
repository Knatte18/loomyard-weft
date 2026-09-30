# Card 11 — loom-recipe-review-rows

**Edit:**
- `contracts/recipes/loom-recipe.yaml`
- `internal/loomshed/loomshed.go`
- `internal/loomshed/interruptpolicy.go`
- `internal/loomcli/wiring.go`
- `internal/loomrecipe/shape_test.go`
- `internal/loomrecipe/coverage_guard_test.go`
- `internal/loomrecipe/recipe_test.go`
- `internal/loomrecipe/fixture_test.go`
- `internal/shedrecipe/coverage_guard_test.go`

**Uses:**
- `internal/shedrecipe/entries_prrework.go`
- `internal/loomengine/rework.go`
- `internal/fabricengine/committedfile.go`
- `internal/websterengine/fingerprint.go`
- `internal/landingshed/rejection.go`

**Intent:**
Wire the two new rows into loom's producer graph (D2, D3, D9).
Names, stated once here: row `PR-Gate` (engine `PRGate`) and row `PR-Rework` (engine `PRRework`), sharing segment `PR-Review`.
In `loom-recipe.yaml`, `Publish.on_done` becomes `PR-Gate`;
`PR-Gate` carries `segment: PR-Review`, `max_bounces: 5`, `on_stuck: PR-Rework`, `on_done: Finalize`;
`PR-Rework` carries `segment: PR-Review`, `on_done: Webster`, no `on_stuck`, and `config: {gate: plan, gate_attempts: 3}` with the same gate comment the other writer rows carry.
Rewrite the header comment's routing audit: Publish still escalates (its failures are merge, verify and push faults nothing in the list produces), the gate bounces to the rework row that produces the diff it guards, `PR-Rework` joins the writer rows whose Stuck halts for a human, the gate's episode Stuck count never resets between rounds so the budget is five rejection rounds per task, and rework cards skip `Plan-Review` because the findings are operator-authored.
Enumerate no row counts, so the next row change does not force a recount.
`loomshed.go` adds `NamePRGate` and `NamePRRework`; `InterruptPolicies` maps both to `InterruptPolicyReinvoke` (the gate spawns nothing, and the rework row's single-session adapter reattaches rather than double-spawns).
`wiring.go` fills `Env.ReworkSpec` from `loomengine.ReworkSpec` exactly as `PlanSpec` is filled, and `Env.Rework` with:
`PlanDir` (`planparser.PlanDir`), `ReworkDir`/`ReworkDirRel` (the loomengine accessors), `ReadCommitted` over `fabricengine.CommittedAnchoredFile(location, rel)`, `ReadRejection` mapping `landingshed.ReadRejection(loomengine.LoomRejectionPath(location))` onto `loomshed.PendingRejection`, `ClearRejection` over `landingshed.RemoveRecord` of the same path, `Commit` over `fabricengine.CommitAnchoredPaths` with the plan directory and the rework directory as the positive pathspec, and `Rebaseline` over `websterengine.RebaselinePlanFingerprint(websterGeom)`.
Every closure opens nothing at wire time, since `wire()` also runs for `status`/`pause`.
Tests: `wantProducerTable` gains both rows with Publish's `onDone` now `NamePRGate`, and `loomRowEngines` maps both;
a new routing test in `recipe_test.go` asserts `Publish.on_done` → `PR-Gate`, the gate's `on_done` → `Finalize` and `on_stuck` → `PR-Rework`, the rework row's `on_done` → `Webster` and empty `on_stuck`, both rows in segment `PR-Review`, and that the main line still ends at `Friction-Reflect`;
`TestRecipe_StructuralCheckHasNoFindings` stays green, confirming `internal/shedcheck` accepts the off-main-line writer row;
the loomrecipe test `Env` fills `ReworkSpec` and `Rework` with fakes;
and `internal/shedrecipe/coverage_guard_test.go` drops the two allowlist entries card 10 added, now that the recipe reaches both engines.

**ImpactSummary:** Reroutes Publish's on_done through the new gate; every other row's routing is unchanged, and in-flight runs migrate as the decision record's auto-mode assumptions state.
