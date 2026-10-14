# Card 5 — darn config on the shared budget family, budget settable per run

**Edit:**
- `internal/darnengine#Config`
- `internal/darnengine#ConfigSpec`
- `internal/darnengine#LoadConfig`
- `internal/darnengine/template.yaml`
- `internal/darnengine/doc.go`
- `internal/darnengine#TestLoadConfig`
- `internal/configreg#Modules`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg#TestModules_LocalLayerRefusals`
- `internal/configreg#TestPrepareRunSettings`
- `internal/loomcli#TestPerchHardCap`
- `internal/loomcli#TestWire_DarnRecipe`
- `contracts/specs/refusal-spec.md`
- `contracts/stencils/orch/orch-template-role.md`
- `docs/overview.md`

**Delete:**
- `internal/darnengine#hardCapSetCommand`
- `internal/darnengine#softCapSetCommand`

**Create:**
- `plan:internal/darnengine#ValidateLocalValues` -> `func ValidateLocalValues(document []byte) error`

**Uses:**
- `plan:internal/reviewbudget#Categories`
- `plan:internal/reviewbudget#Route`
- `plan:internal/reviewbudget#CheckResolved`
- `plan:internal/reviewbudget#CheckLocal`
- `internal/loomrecipe#DarnRouting`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomcli#loomCLI.darnReviewBudget`
- `internal/loomcli#perchHardCap`

**Intent:**
The darn half of D1 and D2 as the record's added entries settle them: darn's `review_max_bounces` and `review_hard_max_bounces` lose `CategoryGuard` and become ordinary keys from `internal/reviewbudget`, so a darn run's local layer, and so its board `run_settings`, sets them.
`review_max_bounces: 0` stays darn's off switch, in any layer: per the "Zero-round perch: the Bouncer settles at once" entry, darn at 0 keeps today's Darn-Write → Merge-In route and skips Describe, so `loomrecipe`'s darn routing, `wireDarn` and the lightweight `darnReviewBudget` stay as they are.
No `skip_review` key, no darn migration and no `PrepareRunSettings` reorder: the settled design drops D3 and D4.
What it lets through: a guard comes off both keys, so a task can switch off its Darn-Review, and with it Describe, through its own `run_settings` or local layer, and can raise or lower its hard cap there; no hub guard bounds that, as "Perch budgets carry no guard" accepts, while Merge-In and the parent lock still run at 0 and a budget above 0 stays bound by a hard cap of at least 1 and the Bouncer's soft-cap rule.

`internal/darnengine`:

- `ConfigSpec` takes `review_max_bounces` and `review_hard_max_bounces` from `reviewbudget.Categories(nil)`, both `CategoryNone`; `verify` stays executable; its doc comment drops the claim that the two budget keys are barred from the local layer.
  `PATTERN-hub-wide-config` already lets darn's local override hold category-none keys, so the module stays `HubWide` and on strict `configengine.Load`.
- `LoadConfig` replaces its two hard-cap refusals with `reviewbudget.CheckResolved(dirs, spec, reviewbudget.Values{RunSoft: cfg.ReviewMaxBounces, RunHard: cfg.ReviewHardMaxBounces})`, which adds `review_max_bounces` below 0 and words every refusal by the layer its value comes from: a value from the local layer names `lyx config darn --layer local --set` in the task worktree and `lyx board upsert`, a hub value `lyx config darn --set` from the prime.
  The hard-below-soft check now applies at a soft cap of 0 as well, where any hard cap of at least 1 passes, so the "while that is above 0" exemption goes.
  `hardCapSetCommand` and `softCapSetCommand` are deleted with it; `LoadConfig` was their one reader.
- `ValidateLocalValues` decodes a local-layer document into a `map[string]any` and returns `reviewbudget.CheckLocal(set, nil)`.
  Its caller has already run `configengine.ValidateLocal` over the same bytes, so a decode failure is a fail-loud guard that wraps the decode error marked `configengine.ErrInvalid`, with no refusal-spec row and no test row.
- `Config`'s `ReviewMaxBounces` comment and `template.yaml`'s two budget comments say the keys are settable per run through `run_settings`, 0 runs neither Darn-Review nor Describe, and the hard cap is at least 1 and not below the budget.
- `doc.go` says the same, and that a budget refusal names the command for the layer its value came from.

`internal/configreg`: `Modules`' darn entry gains `CheckValues: darnengine.ValidateLocalValues`, so a darn `run_settings` budget below its bound is refused at `lyx board upsert` and Seed-Child.

`internal/loomcli`: `perchHardCap`'s darn branch already names its route through `reviewbudget.Route` since card 4, so a hard cap from the run's local layer, which this card admits, names the local route with no code change there.

`contracts/specs/refusal-spec.md`: the build-time row "darn.yaml: review_max_bounces below 0", `loomrecipe`'s `checkDarnReviewBudget`, stays as it is; a load-time row "darn.yaml: review_max_bounces below 0 at load" is added, and the "review_hard_max_bounces below 1" and "hard cap below soft cap" rows say their way forward is by layer, each naming `lyx config darn --layer local --set` and `lyx board upsert` for a value from the local layer and `lyx config darn --set` from the prime for the hub file.
`contracts/stencils/orch/orch-template-role.md`'s run-loop line says the Darn-Review perch runs unless the run's `review_max_bounces` is 0, in `darn.yaml` or the task's `run_settings`.
`docs/overview.md`'s darn paragraph says the budget and hard cap are settable per run through `run_settings`, and `0` routes Darn-Write straight to Merge-In.

Tests:
`TestLoadConfig` keeps its template, review-off, writer and not-initialized rows; its hard-cap-below-1 and budget-above-hard-cap rows keep asserting `lyx config darn --set` and `lyx loom resume`; a `review_max_bounces: -1` row asserts the "0 for no review round" wording.
Its local-override loop keeps the `verify` refusal and replaces the `review_hard_max_bounces` refusal with two subtests: a local `review_max_bounces: 2` and `review_hard_max_bounces: 3` load as budget 2 and hard cap 3, and a local `review_max_bounces: 5` over the hub's hard cap 4 is refused naming the local route with `lyx board upsert` and the hub route `lyx config darn --set`.
`TestModules_EveryTemplateKeyClassified`'s darn `LocalKeys` adds `review_hard_max_bounces` and `review_max_bounces`.
`TestModules_LocalLayerRefusals`' darn row `review_max_bounces: 0` becomes allowed.
`TestPrepareRunSettings`: its row refusing `{"darn": {"review_max_bounces": 0}}` becomes a row accepting it; a row accepts `{"darn": {"review_max_bounces": 2, "review_hard_max_bounces": 3}}`; a row refuses `{"darn": {"review_hard_max_bounces": 0}}` for module darn, naming the key and its bound.
`TestWire_DarnRecipe` gains a subtest that writes a local darn override of `review_max_bounces: 2` and `review_hard_max_bounces: 3` before `wire`, and asserts `c.env.DarnReviewMaxBounces` is 2 and `c.env.RowHardMaxBounces[Darn-Bouncer]` is 3, so the darn run's Darn-Review takes the per-run budget and hard cap.
`TestPerchHardCap` gains a row with a local darn `review_hard_max_bounces: 5` whose raise names `lyx config darn --layer local --set review_hard_max_bounces=<n>` and `lyx board upsert`.

**ImpactSummary:** darn.yaml's two budget keys become ordinary per-run keys checked through reviewbudget at load and at upsert, and darn's hard-cap way forward follows the layer; darn's routing and wiring are unchanged.

**Commit:** 5: darn config: review budget settable per run, refusals by layer
