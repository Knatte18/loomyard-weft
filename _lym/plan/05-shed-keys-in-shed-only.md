# Card 5 — shed-keys-in-shed-only

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#LoadConfig`
- `internal/loomengine#ValidateLocalValues`
- `internal/loomengine#MigrateConfig`
- `internal/loomengine#TestMigrateConfig`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomengine/template.yaml`

- `internal/darnengine#Config`
- `internal/darnengine#ConfigSpec`
- `internal/darnengine#LoadConfig`
- `internal/darnengine#TestLoadConfig`
- `internal/darnengine/template.yaml`
- `internal/darnengine/doc.go`
- `internal/configreg#KeyCopies`
- `internal/configreg#Modules`
- `internal/configreg#PrepareRunSettings`
- `internal/configreg#TestPrepareRunSettings`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/loomcli#loomCLI.seedAndCommitBootstrap`
- `internal/loomcli#loomCLI.arbiterDepsFor`
- `internal/loomcli#tacklerFor`
- `internal/loomcli#TestTacklerFor`
- `internal/battencli#childDriverKind`
- `internal/battencli#TestChildDriverKind`
- `internal/arbiterstop#HoldReading`
- `internal/arbiterstop#TestHoldReading`
- `internal/sheddriver/doc.go`
- `docs/overview.md`
- `PATTERN.md`

**Delete:**
- `internal/loomengine#Config.ShedOverride`
- `internal/loomengine#RecipeOverride`
- `internal/loomengine#migrateShedOverrides`
- `internal/loomengine#retiredShedKeyComments`
- `internal/loomengine#TestMigrateConfig_ShedOverrides`
- `internal/loomengine#TestRecipeOverride`
- `internal/darnengine#Config.ShedOverride`
- `internal/darnengine#ValidateLocalValues`
- `internal/configsync#TestReconcile_CopiesLoomArbiterKeysIntoDarn`

**Uses:**
- `internal/loomengine#TestConfigTemplate_ContainsEveryConfigYAMLTag`
- `internal/configreg/doc.go`
- `internal/arbiterstop/doc.go`
- `internal/configengine#Spec`
- `internal/sheddriver#Override`

**Intent:** `arbiter`, `arbiter_timeout_min`, `arbiter_max_spawns`, `tackler`, `tackler_timeout_min`, `tackler_max_spawns` and `driver_kind` leave `loom.yaml` and `darn.yaml` and live in `shed.yaml` alone (D13); a recipe that needs other values sets them in its profile block under the `shed` module, which lands in the pair's local `shed.yaml` layer like any run setting.

Config modules:
- Both templates drop the seven keys; `loomengine.Config` and `darnengine.Config` drop the seven fields, and both `ShedOverride` methods go. `LoadConfig` of each drops its `sheddriver.CheckOverride` call.
- Each module's `ConfigSpec` drops the seven keys from `Categories` and adds them to `RetiredKeys` with `configengine.CategoryGuard`, never `CategoryNone`, so `ValidateLocal` refuses each in a local file while every load still drops a hub base file's value until the reconcile's `RetiredKeys` prune removes it. Each gets the same `RetiredWayForward` text: `remove the key from the refused local file, profile block or run_settings object, and set it under the shed module instead: in a local file, in the pair's local shed.yaml; in a profile block or run_settings, in the shed object`.
- The keys are never added to `retiredLoomKeys` and darn's `MigrateConfig` never drops them, so no Migrate strips one before a refusal sees it. `MigrateConfig` of loom stops calling `migrateShedOverrides`, which goes with `retiredShedKeyComments`; its doc comment drops the sentence about the four driver and tackler keys.
- `loomengine.ValidateLocalValues` keeps only the `reviewbudget.CheckLocal` budget check. `darnengine.ValidateLocalValues`, which checked only these keys, goes, and darn's entry in `configreg.Modules` drops its `CheckValues`.
- `configreg.KeyCopies` drops the three loom-to-darn `arbiter*` entries and returns an empty list; the copy machinery stays for a later declared copy.

Refusals, both naming the shed module:
- A pair seeded before landing whose local `loom.yaml` or `darn.yaml` holds one of the keys is refused at load by `configengine.Layers`' `ValidateLocal` (card 4), naming the file, the key and the pair's local `shed.yaml`.
- `PrepareRunSettings` checks each registered module's raw object after `moveRunSettings` and before `module.Migrate`: a top-level key the module's `Spec().RetiredWayForward` names is refused as a `*RunSettingsError` marked `configengine.ErrInvalid`, `key %q of module %q is retired; way forward: <text>`, so a board `run_settings` write, every `profiles.yaml` write and Seed-Child, which all go through it, refuse `loom.driver_kind` naming the `shed` module with the same Spec text as the local-file refusal.

Readers: the per-recipe override read goes, and every recipe, pawl included, reads the keys from `shed.yaml` and its layers: `seedAndCommitBootstrap`, `arbiterDepsFor`, `tacklerFor`, `childDriverKind` and `HoldReading` drop their `loomengine.RecipeOverride` call and pass `sheddriver.Override{}` to the resolver they call, keeping `shed.yaml`'s load. `tacklerFor`'s recipe descriptions are unchanged here (card 18).

Tests:
- `TestPrepareRunSettings`: `{"loom":{"driver_kind":"go"}}` and `{"darn":{"arbiter":"opus"}}` are refused, each message ending in exactly the module Spec's `RetiredWayForward` text for the key, the text the local-file refusal carries; `{"shed":{"driver_kind":"go"}}` passes.
- `TestLoadConfig_Refuses` (loom) and `TestLoadConfig` (darn): a local file holding `driver_kind` is refused naming the file, the key and `shed.yaml`; a hub base file holding `arbiter` loads, the value dropped.
- `TestModules_EveryTemplateKeyClassified`'s darn and loom key lists lose the seven keys.
- `TestMigrateConfig` drops its shed-override rows; `TestMigrateConfig_ShedOverrides`, `TestRecipeOverride` and `TestReconcile_CopiesLoomArbiterKeysIntoDarn` go.
- `TestTacklerFor`, `TestChildDriverKind` and `TestHoldReading` stop writing the keys into `loom.yaml`/`darn.yaml` and assert the `shed.yaml` value is taken for a darn, a tapestry and a pawl run alike.
`batten.yaml`'s arbiter and tackler overrides stay. `sheddriver/doc.go` drops `loomengine.RecipeOverride` and says a loom recipe's own values come through its profile block's `shed` object.

`docs/overview.md`'s shed.yaml paragraph drops the sentence that `loom.yaml` and `darn.yaml` override its driver and tackler keys, and names the profile block's `shed` object as a loom recipe's route.

`PATTERN.md`'s `PATTERN-shed-stop-chain` line becomes the declared text:

```
- `PATTERN-shed-stop-chain` — A shed run's stop: rule table, tackler, arbiter, then parent, for every recipe; a stop the driver's arbiter seam marks as judged first, by row and reason, skips the tackler; arbiter keys in shed.yaml, batten's overridable in batten.yaml, a loom recipe's own via its profile block.
```

**ImpactSummary:** Seven keys leave two modules and four readers; five call sites switch to shed.yaml alone, and profiles or local files naming the keys under loom or darn now refuse with the shed way forward.
