```yaml
lens: guard-reach
outcome: reported
```

Lens guard-reach: whether each guard, refusal and test row the cards add can be reached.

**Findings**

1. **MEDIUM — Card 18 (`validate-plan --part`, D7): the "no `PartsPlanWrite` row" refusal can never fire, and its test row reaches a different refusal.**
   - Only `contracts/recipes/tapestry-recipe.yaml` holds an `engine: PlanWrite` row.
   - `refuseVerbOffRecipe` (`internal/loomcli/arm.go:186-198`) already refuses `validate-plan` on every recipe without that row, before the `--part` branch runs.
   - After Card 17, the tapestry recipe also holds the `PartsPlanWrite` row. So every recipe that gets past the plan-verb check has the row, and the `--part` refusal is unreachable.
   - The planned `TestValidatePlanCmd_Part` row "the darn recipe refused" therefore hits the plan-verb refusal ("has no Plan-Write row and so no plan"), not the `--part` refusal it claims to reach.
   - Fix: drop the branch, or route `--part` through `verbRowEngines` to `PartsPlanWrite`. Either way, the test row should assert the refusal that actually fires.

2. **MEDIUM — Cards 1, 9 and 14 (D2): the rule that the chair outlasts its writers is bypassed when `plan_timeout_min` is 0.**
   - `LoadTapestryConfig` accepts 0 as "defer to shuttle's `run_timeout_min`" (`internal/loomengine/tapestry.go`, the zero-timeout comment before the negative check).
   - With `plan_timeout_min: 0` and `parts_chair_timeout_min` unset, `Config.PartsChairTimeout` gives 2×0 = 0. Card 9's rule that a zero seat timeout "takes the table's Timeout" then gives the chair the same deferred shuttle timeout as every writer.
   - A set value such as 10 passes Card 1's "above `plan_timeout_min`" check (10 > 0) but can be shorter than the writers' shuttle default.
   - So the guard holds only when `plan_timeout_min` is non-zero, and no card covers the zero case: neither a refusal nor a resolution against shuttle's `run_timeout_min`.

3. **MEDIUM — Card 7 (`ValidatePart` and the `JoinParts` Go half, D7/D8): the absent-parts-file and malformed-parts-file branches have no test row that reaches them.**
   - Both branches are reachable from a real entry point: `lym loom validate-plan --part`, or `join-parts` after the parts file has been edited or removed.
   - Neither `TestValidatePart`, Card 7's `TestJoinParts` nor Card 18's `TestValidatePlanCmd_Part` lists a row for them, so the `goto --to Parts-Write` and `validate-parts` ways forward are never exercised.

4. **LOW — Card 20 (D1): `segmentWriter`'s walk over Skipped rows can run past the plan writers.**
   - If `Plan-Write` and `Parts-Plan-Write` are both latest-Skipped (flag off when `Parts-Plan-Write` was entered, on when `Plan-Write` was), the walk continues: `Parts-Write` (Skipped), then `Drift-Baseline` (Skipped while drift is off), then `Discussion-Bouncer`.
   - `Discussion-Bouncer` sits outside the Plan-Review segment and its `on_done` names `Drift-Baseline` (`contracts/recipes/tapestry-recipe.yaml`), so the refusal would name `Discussion-Bouncer` as the plan writer.
   - Fix: stop the walk at the first row whose engine is a plan writer, or never step more than once.

5. **LOW — Cards 13, 14 and 16: two planned refusal rows test branches that an upstream check already rules out.**
   - Card 13's "an unknown depth refused" row: `LoadTapestryConfig` refuses any `parts_depth` other than `light` or `medium` (Card 1).
   - Card 14's "a `gates` error fails the table" row: Card 16's `resolvePartGates` validates the `part_gates` list once at construction, so its per-part closure has no production error path.
   - Per `refusal-spec.md`, each should be written as a fail-loud wiring guard rather than a refusal with a reaching test.

6. **LOW — Cards 6 and 7 (D8): the join's exact-handle export match can drift from the part cards after a part check runs.**
   - Card 7's `JoinParts` runs `ValidatePart`, whose resolve pass calls `CanonicalizeHandles`.
   - `CanonicalizeHandles` rewrites each declared draft handle on disk to its canonical `plan:<expected-glyph>` form (`internal/planglyph/handle.go:207-219`).
   - Card 6 then compares exports to Create handles "exactly", against the parts file's draft spelling. Another part's references to that handle are not rewritten, since that part does not declare it.
   - The two spellings usually agree for Go, so this is low risk. But where canonicalization changes a handle, the join raises a false "export not created" refusal, or the spellings drift between parts. No card bounds this.

Each other guard traced reachable: the `join-parts` checks (status `running` is persisted before the producer's `Call`, `internal/shedengine/run.go:310-314`), the writer resume, hold-deferral and join branches, the writer Stuck mapping, the parts-check findings, and the `validate-plan` refusal on darn.

**Tool calls**
- Searched `contracts/recipes` for `engine: PlanWrite` and `internal/shedengine` for `StateRunning`, to learn which recipes hold a Plan-Write row and whether status reads `running` during a call; my context had neither.
- Searched the `StateRunning` assignments in `internal/shedengine`, to find where the running state is persisted.
- Read `internal/shedengine/run.go:295-325`, to confirm `running` is written before the producer's `Call`.
- Read `verbUsesLightweightWiring` (`internal/loomcli/cli.go`), to see whether `validate-parts` gets the paths it needs.
- Read the lightweight wiring's Env fields (`internal/loomcli/wiring.go:273-330`), to confirm `DecisionRecordPath` and `PlanIndex` are filled there.
- Read the `CanonicalizeHandles` contract (`internal/planglyph/handle.go:207`), to check whether the part check rewrites the handle spellings the join compares.
