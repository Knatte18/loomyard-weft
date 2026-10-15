# Card 8 — module-overrides

**Create:**
- `plan:internal/loomengine#Config.ShedOverride` -> `func (c Config) ShedOverride() sheddriver.Override`
- `plan:internal/loomengine#RecipeOverride` -> `func RecipeOverride(dirs configengine.Dirs, recipe string) (sheddriver.Override, error)`
- `plan:internal/loomengine#TestRecipeOverride` -> `func TestRecipeOverride(t *testing.T)`
- `plan:internal/darnengine#Config.ShedOverride` -> `func (c Config) ShedOverride() sheddriver.Override`
- `plan:internal/battenshed#Config.ShedOverride` -> `func (c Config) ShedOverride() sheddriver.Override`
- `plan:internal/battenshed#ValidateLocalValues` -> `func ValidateLocalValues(document []byte) error`
- `plan:internal/battenshed#TestLoadConfig` -> `func TestLoadConfig(t *testing.T)`
- `plan:internal/loomcli#tacklerFor` -> `func tacklerFor(dirs configengine.Dirs, recipe string, reg modelspec.Registry) (sheddriver.TacklerSettings, string, error)`
- `plan:internal/loomcli#TestTacklerFor` -> `func TestTacklerFor(t *testing.T)`
- `plan:internal/battencli#childDriverKind` -> `func childDriverKind(dirs configengine.Dirs, recipe string) (string, error)`
- `plan:internal/battencli#TestChildDriverKind` -> `func TestChildDriverKind(t *testing.T)`

**Delete:**
- `internal/loomengine#ResolveTackler`
- `internal/loomengine#ResolveDriverKind`
- `internal/loomengine#tacklerDefaultModelSpec`
- `internal/loomengine#tacklerDefaultTimeout`
- `internal/loomengine#validateDriverKind`
- `internal/loomengine#validateTacklerKnobs`
- `internal/loomengine#TestResolveTackler`

**Edit:**
- `internal/loomengine/template.yaml`
- `internal/loomengine#Config`
- `internal/loomengine#ValidateLocalValues`
- `internal/loomengine#LoadConfig`
- `internal/loomengine#templateConfig`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/darnengine/template.yaml`
- `internal/darnengine#Config`
- `internal/darnengine#ConfigSpec`
- `internal/darnengine#ValidateLocalValues`
- `internal/darnengine#LoadConfig`
- `internal/darnengine#TestLoadConfig`
- `internal/battenshed/template.yaml`
- `internal/battenshed#Config`
- `internal/battenshed#ConfigSpec`
- `internal/battenshed#LoadConfig`
- `internal/battenshed#SeedChildDeps`
- `internal/battenshed#seedChildProducer.Call`
- `internal/battenshed#battenshedAllowedImports`
- `internal/battenshed#seedChildCalls`
- `internal/battenshed#newSeedChildDeps`
- `internal/battenshed#TestSeedChild_RecipeFromTheBoardTypeAndDriverFromChildDriver`
- `internal/battenshed#TestSeedChild_CancelledDuringASeamErrorReportsTheCancellation`
- `internal/configreg#Modules`
- `internal/configreg#TestPrepareRunSettings`
- `internal/loomcli#loomCLI.goDriverDepsFor`
- `internal/loomcli#loomCLI.seedAndCommitBootstrap`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#Command`
- `internal/battencli#newFakeReceiver`
- `internal/battencli#TestWire_ChildDriverDelegatesToChildDriverOf`
- `internal/shedrecipe#newTestEnvWithSeedChild`
- `internal/testkit/envkit#FullEnv`
- `internal/loomcli/doc.go`
- `contracts/specs/refusal-spec.md`
- `pattern/PATTERN-driver-choice-single-site.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Make `shed.yaml` the defaults and each module's keys its overrides (D5), and have each run resolve its seeded driver and its tackler from its own recipe's module (D5, D6).
The key names stay loom's (D5); `loom-model-key-names` renames them later.

Module keys:
- `loom.yaml` keeps `driver_kind`, `tackler`, `tackler_timeout_min` and `tackler_max_spawns`; `darn.yaml` gains the same four; `batten.yaml` gains the three tackler keys and no `driver_kind`, since batten's own driver is always `go`.
  Each template ships the inherit value, `""`, `""`, `0` and `~`, so a freshly seeded hub does not mask `shed.yaml`.
  Each key's comment says it overrides `shed.yaml` and that empty, 0 or null inherits; `tackler_max_spawns`' says a set 0 turns the tackler off for that module.
  These new comments differ from loom's old ones, which card 9's migration keys on.
- `loomengine.Config.TacklerMaxSpawns` becomes `*int`; `darnengine.Config` and `battenshed.Config` gain the fields their keys need.
  Each module's `ShedOverride` returns its keys as a `sheddriver.Override`, batten's with an empty `DriverKind`.
- `darnengine.ConfigSpec` and `battenshed.ConfigSpec` classify the new keys `configengine.CategoryNone`, open to the local layer and to `run_settings`; darn is hub-wide, so they join its local keys.
- Validation: each module's `LoadConfig` checks its override through `sheddriver.CheckOverride(<module>, cfg.ShedOverride())`, and each module's `ValidateLocalValues` checks a local document's keys through `sheddriver.CheckLocalOverride`.
  loom's `validateDriverKind` and `validateTacklerKnobs` are deleted for it.
  `battenshed.ValidateLocalValues` is new and `configreg.Modules` registers it as batten's `CheckValues`.
  A `driver_kind` in a task's batten `run_settings` or local `batten.yaml` is refused by `configengine.ValidateLocal`, which refuses a key batten's template lacks.
  The base layer keeps keys the template lacks, so `battenshed.LoadConfig` also refuses a resolved document holding a `driver_kind` key, naming `batten config key "driver_kind"`, the reason (batten's own driver is always `go`) and the way forward (remove the key from `batten.yaml`).
  `refusal-spec.md`'s shed section gains that row, "config: driver_kind in batten.yaml", class correctness halt, beside the batten run rows.
  `battenshed` imports `sheddriver` for the override type and the checks, so `battenshedAllowedImports` admits it; `sheddriver` imports no `lyxcwd`, so the told-geometry membership holds.
- `loomengine.RecipeOverride` loads the override of a run's recipe from a task worktree's config layers: `darn.yaml` through `darnengine.LoadConfig` for `shedrun.RecipeDarn`, and `loom.yaml` through `LoadConfig` for `shedrun.RecipeLoom` and the empty recipe.
  It is the one picker both seeding sites and loom's drive wiring use, so loom's choice never leaks into a darn run (D6).

Readers switch over, and `loomengine.ResolveTackler` and `ResolveDriverKind` with their defaults are deleted:
- `seedAndCommitBootstrap` resolves an unseeded run's driver as `sheddriver.ResolveDriverKind(shed, override)`, with the override from `RecipeOverride` for the recipe `resolveSeedRecipe` returns and `shed` from `sheddriver.LoadConfig` over the same config dirs.
  `resolveSeedDriver` still lets a recorded driver win, and the typed `--child-driver` still wins at batten's site (D6).
- `goDriverDepsFor` takes `TacklerDeps.Settings` and `Recipe` from the new `tacklerFor(configDirs, c.recipe, c.registry)`.
  It resolves the settings through `sheddriver.ResolveTackler` over `shed.yaml` and `RecipeOverride` for the recipe, and returns `darnTacklerRecipe` for `shedrun.RecipeDarn` and `loomTacklerRecipe` otherwise, replacing card 4's inline choice, so a darn run takes `darn.yaml`'s keys and line (D5).
  `seedAndCommitBootstrap` needs no such helper: `resolveSeedRecipe` returns `shedrun.RecipeLoom` for every unseeded run (`internal/loomcli/sharedbootstrap.go:184`), and a seeded run's recorded driver wins, so its recipe pick only ever reads `loom.yaml`.
- `battenshed.SeedChildDeps.ChildDriver` becomes `func(recipe string) (string, error)`: Seed-Child passes the recipe it resolved from the board type, `defaultChildRecipe` included.
  battencli's wired closure keeps `childDriverFrom`'s typed `child_driver` precedence, and otherwise calls the new `childDriverKind(fabricengine.ConfigDirs(taskLocation), recipe)` in `internal/battencli/arm.go`, which resolves the kind through `RecipeOverride` and `sheddriver.ResolveDriverKind`, after Seed-Child has written the run settings' local layer.
  The `--child-driver` flag help on run and step says "unset, the child recipe's driver_kind, then shed.yaml's".
  Every fake of the field takes the recipe.
- `loomcli/doc.go`'s seeding sentence, the three `refusal-spec.md` loom config rows (`driver_kind not llm or go`, `tackler not a model-spec`, `tackler knob negative`), the batten row whose way forward ends "to take the task's `driver_kind`" (now the child recipe's `driver_kind`, then `shed.yaml`'s), and `PATTERN-driver-choice-single-site`'s permitted-consumer bullets follow.
  The spec rows name `shed.yaml` and the module files whose keys the message names, with null or empty to inherit.
  The pattern bullets say the two seeding sites read the recipe's own `driver_kind` override, then `shed.yaml`, and that `sheddriver` reads no seed driver field.
  `driverFieldReadCarveOuts` needs no new entry, since neither site reads the seed's `Driver` field outside `loomcli` and the existing carve-out.

Tests:
- `TestRecipeOverride` over temp config layers holding loom's and darn's templates with chosen override values: the loom recipe and the empty recipe read `loom.yaml`'s override, the darn recipe reads `darn.yaml`'s.
  Its rows set different values in the two files, so a picker reading the wrong file fails; the inherit rules stay card 7's tests.
- `TestTacklerFor` over temp config layers: a darn recipe with `darn.yaml` `tackler_max_spawns: 0` and `loom.yaml` 4 resolves 0 spawns and the darn line; with the values swapped it resolves 4; the loom recipe resolves `loom.yaml`'s value and the loom line.
  With 0 spawns the tackler call refuses with `ErrTacklerUnavailable` and spawns nothing (card 4's `TestTacklerCall_SpawnPath`), so this pins the darn acceptance pair at the wiring helper.
- `TestChildDriverKind` over temp config layers: a darn recipe with `darn.yaml` `driver_kind: go` and `loom.yaml` `llm` resolves `go`, the loom recipe resolves `llm`, and empty overrides with `shed.yaml` `go` resolve `go`.
- `templateConfig` and `TestLoadConfig_Loads` take the inherit values and a `tackler_max_spawns: 0` row read as a set 0; `TestLoadConfig_Refuses` keeps its four rows on the new messages, which still carry "llm or go", "must not be negative", "0 for the 10 minute default" and "0 to turn the tackler off".
- darn's `TestLoadConfig` gains rows: the template's override is all-inherit, set values are read, a local `tackler_max_spawns: 0` is read as a set 0, and each invalid value is refused naming `darn config key`.
- `TestLoadConfig` in battenshed: the template's three keys inherit; set values are read; a negative `tackler_max_spawns` is refused naming `batten config key`; a local-layer `batten.yaml` holding `driver_kind: go` is refused naming the key, since `configengine.ValidateLocal` refuses a key the template lacks in the local layer (`internal/configengine/layers.go:170`), the layer a task's `run_settings` writes; a base-layer `batten.yaml` holding `driver_kind: go` is refused by `LoadConfig` naming the key.
- The Seed-Child test records and asserts the recipe `ChildDriver` received, the board's type or `loom` for an empty one.
- `TestPrepareRunSettings` gains rows: a darn `tackler_max_spawns: 0` from `run_settings` becomes a `darn` document; a batten `driver_kind` is refused naming the key; a darn `tackler_max_spawns: -1` is refused through darn's `CheckValues`.

`cmd/lyx/testdata/test-budget.yaml`'s `internal/battencli` row rises from 80 to 81 for `TestChildDriverKind`; battenshed (111 of 114), loomengine (which lost three tests to card 2 and loses `TestResolveTackler` here) and loomcli (which lost twelve to cards 4 and 6) stay inside their rows.

**ImpactSummary:** three module templates and configs gain shed overrides; loom's tackler and driver-kind resolvers are replaced at the two seeding sites and the drive wiring, and Seed-Child's ChildDriver seam takes the recipe across its five fakes.
