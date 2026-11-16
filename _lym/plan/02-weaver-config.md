# Card 2 — weaver-config

**Create:**
- `internal/weaverengine/doc.go`
- `internal/weaverengine/template.yaml`
- `internal/weaverengine/testmain_test.go`
- `plan:internal/weaverengine#configTemplate` -> `var configTemplate string`
- `plan:internal/weaverengine#Config` -> `type Config struct`
- `plan:internal/weaverengine#ConfigSpec` -> `func ConfigSpec() configengine.Spec`
- `plan:internal/weaverengine#ConfigTemplate` -> `func ConfigTemplate() string`
- `plan:internal/weaverengine#ConfigOpenMaps` -> `func ConfigOpenMaps() []string`
- `plan:internal/weaverengine#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs, module string) (Config, error)`
- `plan:internal/weaverengine#ResolveLadder` -> `func ResolveLadder(cfg Config, reg modelspec.Registry) ([]shuttleengine.ModelChoice, error)`
- `plan:internal/weaverengine#TestLoadConfig` -> `func TestLoadConfig(t *testing.T)`
- `plan:internal/weaverengine#TestResolveLadder` -> `func TestResolveLadder(t *testing.T)`

**Edit:**
- `internal/configreg#Modules`

**Extends:**
- `pattern/PATTERN-config-layers.md`
- `//docs/overview.md`

**Uses:**
- `internal/configreg/doc.go`
- `internal/configreg#TestRegistration_MatchesDeclarers`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/websterengine#LoadConfig`

**Intent:** Start `internal/weaverengine` with the `weaver.yaml` config module (D5, D10).

- `template.yaml`, embedded into `configTemplate` with `//go:embed`, holds `ladder` (`sonnet[medium]`, `opus[medium]`, `opus[high]`, a Claude-only default), `fixer_timeout_min: 60` and `verify_gate: webster`, each with a one-line comment.
- `Config` mirrors the keys: `Ladder []string`, `FixerTimeoutMin int`, `VerifyGate string`.
- `ConfigSpec` names the module `weaver` and classifies every key `CategoryNone`; `ConfigOpenMaps` returns `ladder` and the spec carries it as `OpenMaps`, so `lym config weaver --set` carries the list whole, as tapestry's list keys are carried.
- `LoadConfig` loads through `configengine.LoadOrTemplate`, as `websterengine.LoadConfig` does.
  It refuses, marked `configengine.MarkInvalid` and naming the key, a ladder spec `modelspec.Parse` rejects, a non-positive `fixer_timeout_min` and an empty `verify_gate`.
- `ResolveLadder` refuses an empty ladder, and a spec the registry cannot resolve, each naming the key and `lym config weaver` as the way forward.
  Otherwise it returns one `shuttleengine.ModelChoice` per rung, in order, each built by `shuttleengine.NewModelChoice` from the resolved spec.
  The Implementer row calls it when it builds (card 18), so a bad ladder refuses there.
- `configreg.Modules` gains the per-worktree `weaver` row with `Template`, `CheckLoad: checkLoadNamed(weaverengine.LoadConfig, "weaver")`, `OpenMaps` and `Spec`, as `webster`'s row has.
  `TestRegistration_MatchesDeclarers` then sees the new `ConfigTemplate` declarer registered;
  every key is `CategoryNone`, so the module's derived `LocalDenied` is empty, which `TestModules_EveryTemplateKeyClassified` reads as the absent `wantLocalDenied` entry; it needs no new row.
- `pattern/PATTERN-config-layers.md`'s list of degrading callers gains `weaverengine`.
- `docs/overview.md`'s package tree gains the `internal/weaverengine/` line: Weaver, the cold-start implementer with model escalation.
- `doc.go` opens the package doc: Weaver, the `weaver` implementer of tapestry's implementer row, and its `weaver.yaml` keys.
  Later cards extend it.
- `testmain_test.go` is untagged support: `TestMain` runs `tmuxkit.Main`, as `internal/runplan/testmain_test.go` does.

`TestLoadConfig` is tier0: the template loads with the three defaults; a layer file with an unparseable ladder spec, a zero `fixer_timeout_min` and an empty `verify_gate` each refuse naming the key.
`TestResolveLadder` is tier0: the default ladder resolves to three choices in order; an empty ladder and an unknown alias each refuse naming `lym config weaver`.

**ImpactSummary:** Adds one per-worktree config module to the registry's list; no existing module or key changes.
